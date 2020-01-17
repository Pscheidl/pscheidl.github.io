---
title: "Parallel Grid Search in H2O"
published: true
categories:
  - H2O-3
tags:
  - H2O.ai
  - H2O-3
  - Machine learning
  - AI
---

[H<sub>2</sub>O](https://github.com/h2oai/h2o-3) is in it's core a platform for distributed, in-memory computing. On top of the distributed computation platform, the Machine learning algorithms are implemented. At H2O, we design every operation, be it data transformation, training of machine learning models or even parsing to utilize the distributed computation model. In order to work with big data fast, it's necessary. However, a single operation usually can not utilize cluster's computational resources to the very maximum. Data need to be distributed across the cluster, many operations require sequential execution of tasks, which, even if implemented in a distributed manner, follow after each other and require data exchange. These and many other smaller factors, if summed up together, may introduce a significant overhead.

One area of interest is the process of machine learning models training. It takes considerably long for a machine-learning algorithm to build a model. And depending on the algorithm itself, there are the very same inefficiencies to be found. Therefore, **introducing concurrency to model training** might improve utilization of the cluster's resources, mainly CPUs and/or GPUs. One area where lots of models are built is [Grid Search](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/grid-search.html). Grid search "walks" through a space of hyperparameters and builds the respective models. The walking strategy might differ, but they all share the same problem - there are many models to be built. Typically, the build process is constrained by time. And the more models are built, the better.

Until now, H2O, just like other machine learning platforms, has been training one model at a time. Since H2O version `3.28.0.1`, we've given our users a way of building models using Grid Search in parallel. This effectively means there may be `n > 1` models trained on the cluster. While one model is waiting or doing less optimizable calculations, other model can meanwhile utilize cluster's resources. This leads to **more models built in less time**.

## How to use parallel grid search

By default, parallel model building for Grid Search is turned off. This might change in close future. We've introduced a new parameter named `parallelism` for Grid Search. Such parameter, available in Flow, Python and R, is set to `parallelism = 1` by default. This implies models are built sequentially while performing grid search. In order to build models in parallel using grid search, users have two options:

1. Set `parallelism = n` where `n > 1`,
1. Set `parallelism = 0`.

Setting the `parallelism` argument to any number greater than `1` instructs H2O to keep exactly that many models being built at the same time. Until other constraints are hit (time, maximum number of models, etc.). However, setting it to `0` means H2O is free to use internal heuristics to determine the best amount of models to be trained at one time. We call this the **adaptive mode**. Currently, the adaptive mode has no strict contract and the internal heuristics might change. Currently, the adaptive mode assumes the whole cluster consists of machines with equal resources available and simply trains twice more models than the number of CPUs available per node. Given a cluster of `n` nodes, where each node has `c` number of CPUs, the number of model would be `2 * c`. Number of nodes or total number of CPUs available plays no role.

### Python
Parallel grid search can be enabled by simply adding `parallelism = n` to `H2OGridSearch` constructor arguments, where `n = 0` for adaptive mode or `n > 1` for precise control of parallelism. Specifying `n = 1` means sequantial mode.
```
    data = h2o.import_file(path="https://0xdata-public.s3.amazonaws.com/parallel-gs-benchmark/airlines_small.csv")
    hyper_parameters = OrderedDict()
    hyper_parameters["ntrees"] = [1, 5, 50, 100]
    hyper_parameters["max_depth"] = [5, 10, 15]

    grid = H2OGridSearch(H2OGradientBoostingEstimator, hyper_params=hyper_parameters, parallelism=16) # parallelism = 16 means 16 models are built at a time
    grid.train(x=list(range(4)), y=4, training_frame=data)
```

### R

Parallel grid search can be enabled by simply adding `parallelism = n` to `h2o.grid` function arguments, where `n = 0` for adaptive mode or `n > 1` for precise control of parallelism. Specifying `n = 1` means sequantial mode.

```
  data <- h2o.importFile(path = "https://0xdata-public.s3.amazonaws.com/parallel-gs-benchmark/airlines_small.csv")
 
  hyper_parameters = list(ntrees = c(1, 5, 50, 100), learn_rate = c(5, 10, 15))
  grid <- h2o.grid("gbm", grid_id="gbm_grid_test",
                    x=1:4, y=5,
                    training_frame=data,
                    hyper_params = hyper_parameters,
                    parallelism = 16) # parallelism = 16 means 16 models are built at a time
```

## Benchmark

There is a [H2O Parallel Grid Search Benchmark](https://github.com/Pscheidl/h2o-parallel-grid-search-benchmark) available on GitHub. It contains description of experiment, as well as script for reproducibility.

## Limitations and final thoughts

Building lots of models in memory comes at a cost - memory. Every machine learning algorithm requires certain amount of RAM to operate. Building multiple models implies linear, or near-linear growth in memory consumption. From our observations, including the ones in [benchmark](https://github.com/Pscheidl/h2o-parallel-grid-search-benchmark), training multiple models at one time almost always comes with significant performance boost. It's definitely not slower. The smaller (relatively to the cluster's resources) the models are and the faster to build the models are, the more effective is parallel grid search. For super-heavy models (again, this is relative to the cluster's resources), where even training one model at a time barely fits into the memory, using the default `parallelism = 1` might be the only way to train the model. In all other cases, using parallel grid search brings potentially significant advantage.

Even using at least `parallelism = 2` may bring significant speed ups. Memory is the constraint. However, training too many models might introduce large context-switching overhead, which is also unwanted. The current rule for adaptive mode, which spawns `2 * number of leader node's CPUs`, is empirically set for large hyperparameter space, where the model training phase is relatively short. Users are expected to experiment with this number. One of the improvements already planned is to bring a solid adaptive regime that can manage/adapt to cluster's resources on the fly. Stay tuned !

The following people were also involed in the parallel grid search functionality and deserve credit: [Michal Kůrka](https://www.linkedin.com/in/michal-kurka/),[Jan Štěrba](https://www.linkedin.com/in/jansterba/), [Sébastien Poirier](https://www.linkedin.com/in/spoirier/). Remember, H<sub>2</sub>O.ai is open-source and can be found on [GitHub](https://github.com/h2oai/h2o-3). Found a bug ? Head to H<sub>2</sub>O [JIRA](https://www.pavel.cool/images/mojo_import/mojo_import_1.png) and file an issue. Have questions ? H<sub>2</sub>O offers community [Gitter](https://gitter.im/h2oai/h2o-3) and [Slack](https://www.h2o.ai/slack-community/).