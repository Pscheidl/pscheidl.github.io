---
title: "Machine learning with H2O - Hands-on guide for data scientists"
published: true
classes: wide
categories:
  - Machine learning
  - AI
  - H2O-3
tags:
  - H2O.ai
  - H2O-3
  - Machine learning
  - AI
---

H<sub>2</sub>O is the world's number one machine learning platform. It is an open-source software, the [H2O-3 GitHub repository](https://github.com/h2oai/h2o-3) is available for anyone to start hacking. This hands-on guide aims to **explain the basic principles** behind H<sub>2</sub>O and get you as a data scientist **started as quickly as possible** in the most simple way. The rest is just machine learning :)

After reading this guide, you'll be able to

- understand which basic problems does H<sub>2</sub>O solve and why,
- play with H<sub>2</sub>O - explore data, create & tune models,
- see beyond the horizon. Understand where H<sub>2</sub>O  can take you.


As a data scientist, you're most likely to use R and/or Python. H<sub>2</sub>O integrates with both. Interestingly, H<sub>2</sub>O makes it easy to seamlessly switch  Python, R and other data science tools, while still working on the same project. This allows data scientists to interact more easily, as well as using the best tool for the job.  But the possibilities do not stop there. H<sub>2</sub>O also offers it's own web-based interface named [Flow](https://www.h2o.ai/h2o-old/h2o-flow/). By means of Flow, data scientists are able to import, explore and modify datasets, play with models, verify models performance's and much more. Flow is beatiful and quick way to do machine learning. Flows can be saved and given to other data scientists, making cooperation easy.

H<sub>2</sub>O respects habits of data scientists and does not get into their way. Using Python, data scientists are familiar with [Pandas](https://pandas.pydata.org/), [Scikit](http://scikit-learn.org), [numpy](http://www.numpy.org/) and others. H<sub>2</sub>O 's syntax is very similar to those. H<sub>2</sub>O is able to work directly with Pandas' data structures, as well as being compatible with numpy's arrays and primitive Python lists and collections. H<sub>2</sub>O follows the [same pattern with R](http://docs.h2o.ai/h2o/latest-stable/h2o-r/docs/reference/index.html), respecting the naming and syntax R developers are used to. 

## Getting started

The preparations before take-off are short. H<sub>2</sub>O  is extremely easy to start with. All it takes is a common laptop to get started. Once H<sub>2</sub>O  is installed, it is very easy and convenient to import a dataset and create a model out of it. In the examples below, the famous [Airlines Delay](https://www.kaggle.com/giovamata/airlinedelaycauses/data) dataset is used. There is no need to download it, as H<sub>2</sub>O  is going to take care of downloading the dataset for you. The dataset is very intuitive to work with, however if you're unfamiliar with it or simply want to know more, visit [Kaggle website](https://www.kaggle.com/giovamata/airlinedelaycauses/data) for a brief description of each column.

Once fluent in R, working with H<sub>2</sub>O  in Python's environment is easy. And it always works the other way around, as the APIs are very similar, while respecting the target platforms.

### Getting started in R

Open up R CLI (by typing `R` in terminal on most systems) or start R-Studio. It takes just a few lines to install H<sub>2</sub>O  in R.

Before installing H<sub>2</sub>O itself, H<sub>2</sub>O  requires two packages: `RCurl` and `jsonlite`. Install those by entering the following command into R console.

#### Installation
```R
install.packages("RCurl","jsonlite")
```

After `RCurl` and `jsonlite` are installed, one last step is to install H<sub>2</sub>O itself. Installation of latest stable release is done as demonstrated in the following snippet. During the installation, a one-time download of the H<sub>2</sub>O  backend containing all the algorithms and computing know-how will occur.

```R
install.packages("h2o", type="source", repos=(c("http://h2o-release.s3.amazonaws.com/h2o/latest_stable_R")))
```

That's it. H<sub>2</sub>O is now installed and ready to be used. As a first step, it is required to tell R to import the H<sub>2</sub>O library with `library(h2o)` command. Once the library is imported, instruct H<sub>2</sub>O to start itself by calling `h2o.init()`. Both commands are placed in the following code snippet for clarity.

```R
library(h2o)
h2o.init()
```

The `h2o.init()` command is pretty smart and does a lot of things. First, an attempt is made to search for an existing H<sub>2</sub>O instance being started already, before starting a new one. When none is found automatically or specified manually with argument available, a new instance of H<sub>2</sub>O is started. As this is a fresh installation and it is highly unlikely there is an instance of H<sub>2</sub>O already running in your environment, a new instance is started right away. During startup, H<sub>2</sub>O is going to print some useful information. The R version it is running on, H<sub>2</sub>O's version, how to connect to H<sub>2</sub>O's Flow interface or where error logs reside, just to name a few. As usual, an example of H<sub>2</sub>O's output during startup is to be found below this text in the snippet.

```R
> h2o.init()

H2O is not running yet, starting it now...

Note:  In case of errors look at the following log files:
    /tmp/RtmpYs7uDC/h2o_pavel_started_from_r.out
    /tmp/RtmpYs7uDC/h2o_pavel_started_from_r.err

java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

Starting H2O JVM and connecting: . Connection successful!

R is connected to the H2O cluster: 
    H2O cluster uptime:         3 seconds 44 milliseconds 
    H2O cluster timezone:       Europe/Prague 
    H2O data parsing timezone:  UTC 
    H2O cluster version:        3.20.0.2 
    H2O cluster version age:    8 days  
    H2O cluster name:           H2O_started_from_R_pavel_yuw261 
    H2O cluster total nodes:    1 
    H2O cluster total memory:   5.21 GB 
    H2O cluster total cores:    8 
    H2O cluster allowed cores:  8 
    H2O cluster healthy:        TRUE 
    H2O Connection ip:          localhost 
    H2O Connection port:        54321 
    H2O Connection proxy:       NA 
    H2O Internal Security:      FALSE 
    H2O API Extensions:         XGBoost, Algos, AutoML, Core V3, Core V4 
    R Version:                  R version 3.4.4 (2018-03-15) 
 ```

#### Data import
Let's import a dataset and train a model on it very quickly !

```R
airlinesTrainData <- h2o.importFile("https://s3.amazonaws.com/h2o-airlines-unpacked/allyears2k.csv")
```

H<sub>2</sub>O will automatically download the dataset and parse it. It will also try to guess the datatype of each column automatically. H<sub>2</sub>O does a great job at datatype recognition, however each decision can be overridden manually by the user, if required. The imported dataset can also be given a name using `destination_frame` argument. For example `h2o.importFile("https://s3.amazonaws.com/h2o-airlines-unpacked/allyears2k.csv", destination_frame='airlines_train')` imports the very same dataset with airplane delays that can be further addressed by name `airlines_train`, even from other interfaces like Python, another R console, Flow, Java or direct API calls.  If no name is provided, H<sub>2</sub>O will generate an artificial name. Simply put, an imported dataset is called `Frame` in H<sub>2</sub>O. List of frames can be shown by using the `h2o.ls()` function. An example output of calling `h2o.ls()` function can be found in the following code snippet. The very first record in the example is a named frame, the second one is a frame name generated automatically by H<sub>2</sub>O.

```R
> h2o.ls()
                        key
1            airlines_train
2 allyears2k.hex_sid_ae6f_1
```

A preview of the data imported can be displayed with by typing the variable pointing to the `H2OFrame`, in this case `airlinesTrainData`. 


```R
> airlinesTrainData
  Year Month DayofMonth DayOfWeek DepTime CRSDepTime ArrTime CRSArrTime UniqueCarrier FlightNum TailNum ActualElapsedTime CRSElapsedTime AirTime ArrDelay DepDelay
1 1987    10         14         3     741        730     912        849            PS      1451      NA                91             79     NaN       23       11
2 1987    10         15         4     729        730     903        849            PS      1451      NA                94             79     NaN       14       -1
3 1987    10         17         6     741        730     918        849            PS      1451      NA                97             79     NaN       29       11
4 1987    10         18         7     729        730     847        849            PS      1451      NA                78             79     NaN       -2       -1
5 1987    10         19         1     749        730     922        849            PS      1451      NA                93             79     NaN       33       19
6 1987    10         21         3     728        730     848        849            PS      1451      NA                80             79     NaN       -1       -2
  Origin Dest Distance TaxiIn TaxiOut Cancelled CancellationCode Diverted CarrierDelay WeatherDelay NASDelay SecurityDelay LateAircraftDelay IsArrDelayed
1    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN          YES
2    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN          YES
3    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN          YES
4    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN           NO
5    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN          YES
6    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN           NO
  IsDepDelayed
1          YES
2           NO
3          YES
4           NO
5          YES
6           NO
```
#### Model training

On top of the data imported, a model can be built quickly. There are many algorithms available in H<sub>2</sub>O. For the purpose of this tutorial, a widely known [Gradient Boosting Machines](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/gbm.html) method will be used. Let's train a model that is able to predict if the plane arrives late based on month, day of week and distance the plane has to travel before reaching its destination. By invoking `h2o.gbm(...)`, H2O will run a gradient boosting algorithm on the data. There are many variables to play with each and every data scientist can explore on his/her own. Overriding the default hyperparameters would only make this tutorial more complicated. H<sub>2</sub>O only needs to know three things:

- predictor columns,
- response variable column,
- training frame - a dataset to train the model on.

Nothing more. It is even able to guess the distribution of the response variable, eventhough as stated before, everything can be overridden manually by the data scientist, if required. After a model is trained, basic information about the model can be shown just by typing the name of the variable pointing to the trained model, in this case `gbmModel`.

```R
> gbmModel <- h2o.gbm(x=c("Month", "DayOfWeek", "Distance"), y="IsArrDelayed", training_frame = airlinesTrainData)
  |===========================================================================================================================================================| 100%
> gbmModel
Model Details:
==============

H2OBinomialModel: gbm
Model ID:  GBM_model_R_1529834562303_470 
Model Summary: 
  number_of_trees number_of_internal_trees model_size_in_bytes min_depth max_depth mean_depth min_leaves max_leaves mean_leaves
1              50                       50               20591         5         5    5.00000         18         32    27.78000


H2OBinomialMetrics: gbm
** Reported on training data. **

MSE:  0.2349663
RMSE:  0.4847332
LogLoss:  0.6609351
Mean Per-Class Error:  0.4892883
AUC:  0.6237765
Gini:  0.2475531

Confusion Matrix (vertical: actual; across: predicted) for F1-optimal threshold:
        NO   YES    Error          Rate
NO     604 18933 0.969084  =18933/19537
YES    232 24209 0.009492    =232/24441
Totals 836 43142 0.435786  =19165/43978

Maximum Metrics: Maximum metrics at their respective thresholds
                        metric threshold    value idx
1                       max f1  0.431588 0.716423 368
2                       max f2  0.355217 0.862241 395
3                 max f0point5  0.513454 0.633681 278
4                 max accuracy  0.511993 0.594661 279
5                max precision  0.972534 1.000000   0
6                   max recall  0.347469 1.000000 397
7              max specificity  0.972534 1.000000   0
8             max absolute_mcc  0.605839 0.175948 151
9   max min_per_class_accuracy  0.539888 0.582464 234
10 max mean_per_class_accuracy  0.547177 0.584595 225

Gains/Lift Table: Extract with `h2o.gainsLift(<model>, <data>)` or `h2o.gainsLift(<model>, valid=<T/F>, xval=<T/F>)`
```

Overall, this model is not expected to perform very well, given the huge error rate to be observed in the confusion matrix. By playing with different [GBM hyperparameters](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/gbm.html#defining-a-gbm-model) and including different predictors into the model, much better results can be achieved. As a data scientist, the task of making the model perform better is easy for you, that's certain. In the `h2o.` package, there are many additional functions to work with the model and help a data scientist understand what happened during the training phase much more. As an example, the `h2o.varimp` function shows importances of variables (relative, percentage) taken into account in the model. As you begin exploring H<sub>2</sub>O, the [reference guide](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science.html) will guide you through all the H<sub>2</sub>O's functionality.

```R
> h2o.varimp(gbmModel)
Variable Importances: 
   variable relative_importance scaled_importance percentage
1  Distance         1379.994019          1.000000   0.501313
2     Month          970.739441          0.703437   0.352643
3 DayOfWeek          402.024353          0.291323   0.146044
```

Looks like distance is much more important than month or day of week when it comes to plane being delayed. Of course, according to this very basic model given the default parameters. A pro-tip at the end: H<sub>2</sub>O supports XGBoost. It is trivial to swap GBM for XGBoost in this phase and see how the model changes with default hyperparameters:

```R
xgBoostModel <- h2o.xgboost(x=c("Month", "DayOfWeek", "Distance"), y="IsArrDelayed", training_frame = airlinesTrainData)
```

#### Prediction

Prediction is very simple as well. Calling `h2o.predict(model, data)`, where `model` is the variable pointing to the model trained and `data` is the `H2OFrame` with data to do the prediction on. To test the prediction is functional in a very simple way, let's use the `gbmModel` and let it predict the original training dataset.

```R
> h2o.predict(gbmModel, airlinesTrainData)
  |===========================================================================================================================================================| 100%
  predict        NO       YES
1     YES 0.1419937 0.8580063
2     YES 0.1015739 0.8984261
3     YES 0.2036055 0.7963945
4     YES 0.1239904 0.8760096
5     YES 0.1384360 0.8615640
6     YES 0.1419937 0.8580063

[43978 rows x 3 columns] 
```

The prediction is not very accurate in case there was no delay. This is expected, as the model is very basic. Of course, the confusion matrix seen earlier in this tutorial gave out the information about such "bad" performance beforehand.

### Getting started in Python

In order to get started in Python, only few lines of code are required. A common way of installing dependencies in python is `pip` or `anaconda`. In this tutorial, `pip` is preferred due to most user being familiar with it. If you'd like to use Conda, please follow the tutorial in [H2O documentation](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/downloading.html#install-on-anaconda-cloud). Python 2.7 up to Python 3.x are supported. The differences are minimal and this guide should work on both versions.

#### Installation

Before installing H<sub>2</sub>O itself, a few dependencies are required. Please install them using `pip install`. On some systems, super-user privileges may be required. If so, adding `sudo` before `pip install` will solve the problem.

```python
pip install requests
pip install tabulate
pip install scikit-learn
pip install colorama
pip install future
```

Once the dependencies required are installed, one last step is to install H<sub>2</sub>O itself. Installation of latest stable release is done as demonstrated in the following snippet. During the installation, a one-time download of the H<sub>2</sub>O backend containing all the algorithms and computing know-how will occur.

```python
pip install -f http://h2o-release.s3.amazonaws.com/h2o/latest_stable_Py.html h2o
```

That's it. H<sub>2</sub>O is now installed and ready to be used. As a first step, it is required to tell Python to import the H<sub>2</sub>O module with `import h2o` command. Once the module is imported, instruct H<sub>2</sub>O to start itself by calling `h2o.init()`. Both commands are placed in the following code snippet for clarity. The process of setup is very similar to R.

```python
import h2o
h2o.init()
```

The `h2o.init()` command is pretty smart and does a lot of things. First, an attempt is made to search for an existing H<sub>2</sub>O instance being started already, before starting a new one. When none is found automatically or specified manually with argument available, a new instance of H<sub>2</sub>O is started. As this is a fresh installation and it is highly unlikely there is an instance of H<sub>2</sub>O already running in your environment, a new instance is started right away. During startup, H2O is going to print some useful information. Version of the Python it is running on, H<sub>2</sub>O's version, how to connect to H2O's Flow interface or where error logs reside, just to name a few. As usual, an example of H<sub>2</sub>O's output during startup is to be found below this text in the snippet.

```text
>>> h2o.init()
Checking whether there is an H2O instance running at http://localhost:54321..... not found.
Attempting to start a local H2O server...
  Java Version: java version "1.8.0_171"; Java(TM) SE Runtime Environment (build 1.8.0_171-b11); Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
  Starting server from /usr/local/lib/python2.7/dist-packages/h2o/backend/bin/h2o.jar
  Ice root: /tmp/tmp7R8OvB
  JVM stdout: /tmp/tmp7R8OvB/h2o_pavel_started_from_python.out
  JVM stderr: /tmp/tmp7R8OvB/h2o_pavel_started_from_python.err
  Server is running at http://127.0.0.1:54321
Connecting to H2O server at http://127.0.0.1:54321... successful.
versionFromGradle='3.19.0',projectVersion='3.19.0.99999',branch='pavel_pubdev-5336',lastCommitHash='4e976bde05d5096d31a5889a340af56ae256c8c0',gitDescribe='jenkins-master-4235-5-g4e976bd',compiledOn='2018-03-18 16:38:51',compiledBy='pavel'
--------------------------  ----------------------------------------
H2O cluster uptime:         01 secs
H2O cluster timezone:       Europe/Prague
H2O data parsing timezone:  UTC
H2O cluster version:        3.19.0.99999
H2O cluster version age:    3 months and 5 days
H2O cluster name:           H2O_from_python_pavel_oy9g50
H2O cluster total nodes:    1
H2O cluster free memory:    5.207 Gb
H2O cluster total cores:    8
H2O cluster allowed cores:  8
H2O cluster status:         accepting new members, healthy
H2O connection url:         http://127.0.0.1:54321
H2O connection proxy:
H2O internal security:      False
H2O API Extensions:         XGBoost, Algos, AutoML, Core V3, Core V4
Python version:             2.7.15 candidate
--------------------------  ----------------------------------------

 ```

#### Data import
Let's import a dataset and train a model on it very quickly !

```python
airlines_train_data = h2o.import_file("https://s3.amazonaws.com/h2o-airlines-unpacked/allyears2k.csv")
```

H<sub>2</sub>O will automatically download the dataset and parse it. It will also try to guess the datatype of each column automatically. H<sub>2</sub>O does a great job at datatype recognition, however each decision can be overridden manually by the user, if required. The imported dataset can also be given a name using `destination_frame` argument. For example `h2o.import_file("https://s3.amazonaws.com/h2o-airlines-unpacked/allyears2k.csv", destination_frame='airlines_train')` imports the very same dataset with airplane delays that can be further addressed by name `airlines_train`, even from other interfaces like Python, another R console, Flow, Java or direct API calls.  If no name is provided, H<sub>2</sub>O will generate an artificial name. Simply put, an imported dataset is called `Frame` in H<sub>2</sub>O. List of frames can be shown by using the `h2o.ls()` function. An example output of calling `h2o.ls()` function can be found in the following code snippet. The very first record in the example is a named frame, the second one is a frame name generated automatically by H<sub>2</sub>O.
```python
>>> h2o.ls()
              key
0  airlines_train
1  allyears2k.hex
```

A preview of the data imported can be displayed with by typing the variable pointing to the H<sub>2</sub>O Frame, in this case `airlines_train_data`. 


```python
>>> airlines_train_data
  Year Month DayofMonth DayOfWeek DepTime CRSDepTime ArrTime CRSArrTime UniqueCarrier FlightNum TailNum ActualElapsedTime CRSElapsedTime AirTime ArrDelay DepDelay
1 1987    10         14         3     741        730     912        849            PS      1451      NA                91             79     NaN       23       11
2 1987    10         15         4     729        730     903        849            PS      1451      NA                94             79     NaN       14       -1
3 1987    10         17         6     741        730     918        849            PS      1451      NA                97             79     NaN       29       11
4 1987    10         18         7     729        730     847        849            PS      1451      NA                78             79     NaN       -2       -1
5 1987    10         19         1     749        730     922        849            PS      1451      NA                93             79     NaN       33       19
6 1987    10         21         3     728        730     848        849            PS      1451      NA                80             79     NaN       -1       -2
  Origin Dest Distance TaxiIn TaxiOut Cancelled CancellationCode Diverted CarrierDelay WeatherDelay NASDelay SecurityDelay LateAircraftDelay IsArrDelayed
1    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN          YES
2    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN          YES
3    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN          YES
4    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN           NO
5    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN          YES
6    SAN  SFO      447    NaN     NaN         0               NA        0          NaN          NaN      NaN           NaN               NaN           NO
  IsDepDelayed
1          YES
2           NO
3          YES
4           NO
5          YES
6           NO
```

#### Model training

On top of the data imported, a model can be built quickly. There are many algorithms available in H2O. For the purpose of this tutorial, a widely known [Gradient Boosting Machines](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/gbm.html) method will be used. Let's train a model that is able to predict if the plane arrives late based on month, day of week and distance the plane has to travel before reaching its destination. GBM resides in `h2o.estimators.gbm` package. First step is to import `H2OGradientBoostingEstimator` to avoid the need for typing a fully qualified name in future.

```python
from h2o.estimators.gbm import H2OGradientBoostingEstimator
```

First, it is required to construct a new GBM estimator instance by calling `gbm_model = H2OGradientBoostingEstimator()` constructor. By invoking `gbm_model.train(...)`, H<sub>2</sub>O will run a gradient boosting algorithm on the data. There are many variables to play with each and every data scientist can explore on his/her own. Overriding the default hyperparameters would only make this tutorial more complicated. H2O only needs to know three things:

- predictor columns,
- response variable column,
- training frame - a dataset to train the model on.


```python
gbm_model = H2OGradientBoostingEstimator()
gbm_model.train(x = ["Month", "DayOfWeek", "Distance"], y = "IsArrDelayed", training_frame=airlines_train_data)
```

Nothing more. It is even able to guess the distribution of the response variable, eventhough as stated before, everything can be overridden manually by the data scientist, if required. After a model is trained, basic information about the model can be shown just by typing the name of the variable pointing to the trained model, in this case `gbm_model`. In the next figure, there is a demonstration of previous steps regarding model training, with H<sub>2</sub>O's output included.

```text
>>> from h2o.estimators.gbm import H2OGradientBoostingEstimator
>>> gbm_model = H2OGradientBoostingEstimator();
>>> gbm_model.train(x = ["Month", "DayOfWeek", "Distance"], y = "IsArrDelayed", training_frame=airlines_train_data)
gbm Model Build progress: |███████████████████████████████████████████████████████████████████| 100%
>>> gbm_model
Model Details
=============
H2OGradientBoostingEstimator :  Gradient Boosting Machine
Model Key:  GBM_model_python_1529844691141_1


ModelMetricsBinomial: gbm
** Reported on train data. **

MSE: 0.234966307798
RMSE: 0.484733233643
LogLoss: 0.660935092712
Mean Per-Class Error: 0.41540494848
AUC: 0.623776525749
Gini: 0.247553051497
Confusion Matrix (Act/Pred) for max f1 @ threshold = 0.43158830279: 
       NO    YES    Error    Rate
-----  ----  -----  -------  -----------------
NO     604   18933  0.9691   (18933.0/19537.0)
YES    232   24209  0.0095   (232.0/24441.0)
Total  836   43142  0.4358   (19165.0/43978.0)
Maximum Metrics: Maximum metrics at their respective thresholds

metric                       threshold    value     idx
---------------------------  -----------  --------  -----
max f1                       0.431588     0.716423  368
max f2                       0.355217     0.862241  395
max f0point5                 0.513454     0.633681  278
max accuracy                 0.511993     0.594661  279
max precision                0.972534     1         0
max recall                   0.347469     1         397
max specificity              0.972534     1         0
max absolute_mcc             0.605839     0.175948  151
max min_per_class_accuracy   0.539888     0.582464  234
max mean_per_class_accuracy  0.547177     0.584595  225
Gains/Lift Table: Avg response rate: 55,58 %

    group    cumulative_data_fraction    lower_threshold    lift      cumulative_lift    response_rate    cumulative_response_rate    capture_rate    cumulative_capture_rate    gain      cumulative_gain
--  -------  --------------------------  -----------------  --------  -----------------  ---------------  --------------------------  --------------  -------------------------  --------  -----------------
    1        0.0100732                   0.897096           1.70593   1.70593            0.948081         0.948081                    0.0171842       0.0171842                  70.5933   70.5933
    2        0.0221247                   0.864273           1.64658   1.6736             0.915094         0.930113                    0.0198437       0.0370279                  64.6578   67.3602
    3        0.0302651                   0.83021            1.54302   1.63848            0.857542         0.910594                    0.0125609       0.0495888                  54.3021   63.848
    4        0.040111                    0.753167           1.40873   1.58208            0.78291          0.879252                    0.0138701       0.0634589                  40.8732   58.2085
    5        0.0514803                   0.696079           1.34952   1.53072            0.75             0.850707                    0.0153431       0.078802                   34.9515   53.0722
    6        0.103688                    0.649372           1.30093   1.41502            0.722997         0.786404                    0.0679187       0.146721                   30.0926   41.5018
    7        0.150348                    0.618087           1.225     1.35605            0.680799         0.75363                     0.0571581       0.203879                   22.4998   35.6046
    8        0.20101                     0.600111           1.17103   1.30942            0.650808         0.727715                    0.0593265       0.263205                   17.1034   30.9416
    9        0.3004                      0.578542           1.07854   1.23303            0.599405         0.685262                    0.107197        0.370402                   7.85418   23.3029
    10       0.400632                    0.557964           1.03234   1.18282            0.57373          0.657359                    0.103474        0.473876                   3.23424   18.282
    11       0.500568                    0.540445           1.00633   1.14758            0.559272         0.637776                    0.100569        0.574445                   0.632788  14.7584
    12       0.606894                    0.525153           0.973175  1.11703            0.540847         0.620794                    0.103474        0.677918                   -2.68253  11.7028
    13       0.701874                    0.508853           0.921     1.0905             0.511851         0.606052                    0.087476        0.765394                   -7.89998  9.05014
    14       0.800673                    0.492089           0.869653  1.06325            0.483314         0.590907                    0.0859212       0.851315                   -13.0347  6.32497
    15       0.90261                     0.465819           0.80997   1.03465            0.450145         0.575009                    0.0825662       0.933882                   -19.003   3.46453
    16       1                           0.279442           0.678906  1                  0.377306         0.555755                    0.0661184       1                          -32.1094  0

Scoring History: 
     timestamp            duration    number_of_trees    training_rmse    training_logloss    training_auc    training_lift    training_classification_error
---  -------------------  ----------  -----------------  ---------------  ------------------  --------------  ---------------  -------------------------------
     2018-06-24 15:03:17  0.087 sec   0.0                0.49688163904    0.686916957642      0.5             1.0              0.444244849698
     2018-06-24 15:03:17  0.368 sec   1.0                0.495487418342   0.684101672491      0.583881451988  1.61617611229    0.444244849698
     2018-06-24 15:03:18  0.487 sec   2.0                0.494360221437   0.681803341309      0.584731061951  1.65529653938    0.444244849698
     2018-06-24 15:03:18  0.571 sec   3.0                0.493435134409   0.679892007692      0.584823366972  1.68638964557    0.444244849698
     2018-06-24 15:03:18  0.659 sec   4.0                0.492679667171   0.678305979102      0.585117301795  1.65729931801    0.444244849698
---  ---                  ---         ---                ---              ---                 ---             ---              ---
     2018-06-24 15:03:19  1.667 sec   46.0               0.485057037114   0.661619456851      0.622140956624  1.70738658629    0.435422256583
     2018-06-24 15:03:19  1.688 sec   47.0               0.484945381131   0.661377063627      0.622823558497  1.70738658629    0.435604165719
     2018-06-24 15:03:19  1.710 sec   48.0               0.484866365041   0.661216334425      0.623275804097  1.70738658629    0.435877029424
     2018-06-24 15:03:19  1.734 sec   49.0               0.484810585256   0.661098923553      0.623470282123  1.70738658629    0.435604165719
     2018-06-24 15:03:19  1.760 sec   50.0               0.484733233643   0.660935092712      0.623776525749  1.70593338378    0.435786074856

See the whole table with table.as_data_frame()
Variable Importances: 
variable    relative_importance    scaled_importance    percentage
----------  ---------------------  -------------------  ------------
Distance    1379.99                1                    0.501313
Month       970.739                0.703437             0.352643
DayOfWeek   402.024                0.291323             0.146044

```

Overall, this model is not expected to perform very well, given the huge error rate to be observed in the confusion matrix. By playing with different [GBM hyperparameters](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science/gbm.html#defining-a-gbm-model) and including different predictors into the model, much better results can be achieved. As a data scientist, the task of making the model perform better is easy for you, that's certain. To get detailed information about model and its scoring history, invoke the `print(gbm_model)` command. The output contains table with importances of variables (relative, percentage, scaled) taken into account in the model. Also, a detailed scoring history is available, as well as basic measures like mean squared error (MSE). As you begin exploring H<sub>2</sub>O, the [reference guide](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science.html) will guide you through all the H<sub>2</sub>O's functionality. A shortened example of a detailed view on GBM model is to be found in the next figure. Some of the text was omitted.

```text
print(gbm_model)
Model Details
=============
H2OGradientBoostingEstimator :  Gradient Boosting Machine
Model Key:  GBM_model_python_1529844691141_1


ModelMetricsBinomial: gbm
** Reported on train data. **

Scoring History: 
     timestamp            duration    number_of_trees    training_rmse    training_logloss    training_auc    training_lift    training_classification_error
---  -------------------  ----------  -----------------  ---------------  ------------------  --------------  ---------------  -------------------------------
     2018-06-24 15:03:17  0.087 sec   0.0                0.49688163904    0.686916957642      0.5             1.0              0.444244849698
     2018-06-24 15:03:17  0.368 sec   1.0                0.495487418342   0.684101672491      0.583881451988  1.61617611229    0.444244849698
     2018-06-24 15:03:18  0.487 sec   2.0                0.494360221437   0.681803341309      0.584731061951  1.65529653938    0.444244849698
     2018-06-24 15:03:18  0.571 sec   3.0                0.493435134409   0.679892007692      0.584823366972  1.68638964557    0.444244849698
     2018-06-24 15:03:18  0.659 sec   4.0                0.492679667171   0.678305979102      0.585117301795  1.65729931801    0.444244849698
---  ---                  ---         ---                ---              ---                 ---             ---              ---
     2018-06-24 15:03:19  1.667 sec   46.0               0.485057037114   0.661619456851      0.622140956624  1.70738658629    0.435422256583
     2018-06-24 15:03:19  1.688 sec   47.0               0.484945381131   0.661377063627      0.622823558497  1.70738658629    0.435604165719
     2018-06-24 15:03:19  1.710 sec   48.0               0.484866365041   0.661216334425      0.623275804097  1.70738658629    0.435877029424
     2018-06-24 15:03:19  1.734 sec   49.0               0.484810585256   0.661098923553      0.623470282123  1.70738658629    0.435604165719
     2018-06-24 15:03:19  1.760 sec   50.0               0.484733233643   0.660935092712      0.623776525749  1.70593338378    0.435786074856

See the whole table with table.as_data_frame()
Variable Importances: 
variable    relative_importance    scaled_importance    percentage
----------  ---------------------  -------------------  ------------
Distance    1379.99                1                    0.501313
Month       970.739                0.703437             0.352643
DayOfWeek   402.024                0.291323             0.146044

```

Looks like distance is much more important than month or day of week when it comes to plane being delayed. Of course, according to this very basic model given the default parameters. A pro-tip at the end: H<sub>2</sub>O supports XGBoost. It is trivial to swap GBM for XGBoost in this phase and see how the model changes with default hyperparameters:

```python
from h2o.estimators.xgboost import H2OXGBoostEstimator
xgb_model.train(x = ["Month", "DayOfWeek", "Distance"], y = "IsArrDelayed", training_frame=airlines_train_data)
```

#### Prediction

Once a model is created, predictions are simply done by calling `predict(data)` method on a model, where the `data` argument is the variable pointing to an `H2OFrame` with data to do the prediction on. To test the prediction is functional in a very simple way, let's use the `gbm_model` and let it predict the original training dataset by issuing `gbm_model.predict(airlines_train_data)` command.

```text
>>> gbm_model.predict(airlines_train_data)
gbm prediction progress: |████████████████████████████████████████████████████████████████████| 100%
predict          NO       YES
---------  --------  --------
YES        0.141994  0.858006
YES        0.101574  0.898426
YES        0.203606  0.796394
YES        0.12399   0.87601
YES        0.138436  0.861564
YES        0.141994  0.858006
YES        0.101574  0.898426
YES        0.103421  0.896579
YES        0.203606  0.796394
YES        0.12399   0.87601

[43978 rows x 3 columns]

```

A result of the prediction is a `H2OFrame`. Pointer to it can be saved into a variable as well, e.g. `prediction = gbm_model.predict(airlines_train_data)`. The table printed is only a preview of the first few predictions made. As the above example demonstrates, the prediction is not very accurate in case there was no delay. This is expected, as the model is very basic. Of course, the confusion matrix seen earlier in this tutorial gave out the information about such "bad" performance beforehand.



### Getting started with Flow

Flow is H<sub>2</sub>O's web interface. It is very powerful and oriented on visuals. It has it all. From fast prototyping of new models, visualizing and refining existing achievements to scoring.
<a href="/assets/images/2018-06-24-h2o-3-introduction/flow.png"><img src="/assets/images/2018-06-24-h2o-3-introduction/flow.png"></a>

Flow is active whenever H<sub>2</sub>O is started. If you've followed previous Python or R tutorials, during the active Python or R session, Flow can be reached from your browser. Whenever `h2o.init()` is called, Flow is started with H<sub>2</sub>O. In the output after `h2o.init()`, the URL and are printed. When ran locally, the Flow is usually bound to `localhost`, with the default port being `54321`. Address or port may be changed by `h2o.init()` arguments, e.g. `h2o.init(ip='127.0.0.1', port='10001')`.

```text
>>> h2o.init()
Checking whether there is an H2O instance running at http://localhost:54321..... not found.
Attempting to start a local H2O server...
//Some output omitted
--------------------------  ----------------------------------------
H2O cluster uptime:         01 secs
H2O cluster timezone:       Europe/Prague
H2O data parsing timezone:  UTC
H2O cluster version:        3.19.0.99999
H2O cluster version age:    3 months and 5 days
H2O cluster name:           H2O_from_python_pavel_oy9g50
H2O cluster total nodes:    1
H2O cluster free memory:    5.207 Gb
H2O cluster total cores:    8
H2O cluster allowed cores:  8
H2O cluster status:         accepting new members, healthy
H2O connection url:         http://127.0.0.1:54321            <<<<--- URL to FLOW
H2O connection proxy:
H2O internal security:      False
H2O API Extensions:         XGBoost, Algos, AutoML, Core V3, Core V4
Python version:             2.7.15 candidate
--------------------------  ----------------------------------------

 ```

From the shortened output, the actual URL can be observed: `H2O connection url: http://127.0.0.1:54321`. As `127.0.0.1` resolves to localhost, copying the given URL or simply typing `localhost:54321` into a web browser opens H2O Flow.

#### Starting Flow without Python/R

There is no need to install the Python module or R library in order to run H<sub>2</sub>O. Just download the [latest stable H2O release](http://h2o-release.s3.amazonaws.com/h2o/latest_stable.html) and run it. Java is required to run H<sub>2</sub>O.

1. Download package with latest stable release
1. Unpack it
1. Run java -jar h2o.jar

The `h2o.jar` file is to be found in the extracted directory. If you're on Windows, double-clicking the h2o.jar file should be sufficient to run it, if there is Java installed. At the time this tutorial is written, Java version 7 is the minimum required version. Java 8 is recommended.


#### Importing Data with Flow

On top of the page, click on `Data`. As the menu appears, choose `Import files`. Do not interchange with "Upload file" option, which is only useful to upload files from your very own machine. After clicking on the `Import files` option, a form appears at the bottom of the page. Please fill the `Search` input box with `https://s3.amazonaws.com/h2o-airlines-unpacked/allyears2k.csv` to download the Airlines dataset described at the beginning of this chapter. By clicking on the `magnifying glass` next to the input box, H2O is going to verify it can reach the file. This way, even local files or files from remote filesystems (including Hadoop filesystem) can be imported. By inserting whole folders, H2O will pop-up all the files available, thus enabling multi-file import with ease. In this simple case, after clicking on the `magnifying glass`, a single file named exactly as the URL inserted into the input box will appear a little bit lower. By clicking on the `plus icon`, the file is marked for import. By clicking `Import button`, H2O will automatically download the file.

After the file is imported, another form will appear. This time, H<sub>2</sub>O confirms the file has been imported and asks to user to starting parsing it. By clicking on `Parse these files`, H2O will begin parsing.

<a href="/assets/images/2018-06-24-h2o-3-introduction/flow_import.png"><img src="/assets/images/2018-06-24-h2o-3-introduction/flow_import.png"></a>


However, H2O is not going to parse the whole file right away. The could be potentially very big (or there may be multiple files). Instead, it will automatically detect the format and suggests best parsing settings based on internal heuristics. In case of CSV imported in this tutorial, it will correctly detect the format, set separator to be a comma, detect column headers on first row. However, there is more - columns datatypes are detected. Yet the data scientist has the power to override every decision, e.g. change column type. By clicking on `Parse` button at the very bottom, the file is finally parsed by H<sub>2</sub>O.

<a href="/assets/images/2018-06-24-h2o-3-introduction/flow_parse.png"><img src="/assets/images/2018-06-24-h2o-3-introduction/flow_parse.png"></a>

The sample dataset is very small (4,4 MB) and H<sub>2</sub>O's CSV parser is very fast, so the parsing should be quick. After the parsing phase, H<sub>2</sub>O will inform you about the dataset being ready for an inspection. By clicking on the `View` button, a table view preview of the parsed data appears. Here, the user has several choices, including viewing all the data, exporting it, splitting it into several smaller chunks (e.g.for testing), or most importantly to `Build Model`.

<a href="/assets/images/2018-06-24-h2o-3-introduction/flow_data.png"><img src="/assets/images/2018-06-24-h2o-3-introduction/flow_data.png"></a>

By clicking on the `Build model` button above the data overview, a dialogue asking the user to select an algorithm appears. In this tutorial, GBM is used as a simple example, therefore select `Gradient Boosting Machine`. As with Python and R, let's train a model that is able to predict if the plane arrives late based on month, day of week and distance the plane has to travel before reaching its destination. From top to bottom, only few of the choices need your attention right now. Most of these are hyperparameters, which are of course important in real world, yet for a getting started guide, the default values will suffice.

The `training_frame` option is already pre-set by flow to point to the data imported. Creating validation frames from training is easy using the following option, which we'll leave intact for now. Since this model is trying to predict if the plane will be delayed on arrival ot its destination, the response column should be set to `IsArrDelayed`. The prediction is based on month, day of week and traveling distance (chosen arbitrarily). Therefore, in the ignored column section, first check all the columns as ignored and leave only `Month`, `DayOfWeek`, `Distance` and `IsArrDelayed` (response variable) unchecked.

After the parameters are set, click on `Build model` button at the bottom to start the model training.

<a href="/assets/images/2018-06-24-h2o-3-introduction/flow_model.png"><img src="/assets/images/2018-06-24-h2o-3-introduction/flow_model.png"></a>


Model training in this small case should be done in an instant. H<sub>2</sub>O will again display a message about training progress and after the training is finished, the model can be displayed by clicking on the `View` button. A complete model description appears. Variable importance, logloss, complete history per each round of training (including durations) & much more can be observed.

<a href="/assets/images/2018-06-24-h2o-3-introduction/flow_model_trained.png"><img src="/assets/images/2018-06-24-h2o-3-introduction/flow_model_trained.png"></a>


#### Predicting with Flow

After the model is built, click the `Predict` button (with a lightning icon) on top of model summary reached in previous steps. To test the prediction is functional in a very simple way, let's use the current model and let it predict the original training dataset. After clicking on the `Predict` button, select the training frame in the dropdown menu's `Frame` option and click on `Predict`. The resulting predictions appear, including various metrics. The model is not very accurate, especially the error rate regarding prediction of a plane not arriving late are very inaccurate. Creating a better model by tuning the hyperparameters or playing with various predictors is up to each and every data scientist to try now.

<a href="/assets/images/2018-06-24-h2o-3-introduction/flow_predict.png"><img src="/assets/images/2018-06-24-h2o-3-introduction/flow_predict.png"></a>

## Where to go next ?

In this hands-on guide, very basic ways to interact with H<sub>2</sub>O were shown. However, there is so much more to H<sub>2</sub>O. You can take H<sub>2</sub>O from your laptop to large clusters, creating models and predicting on extremely large datasets. Cross-validation, stacked ensembled, tuning parameters with grid search and many, many other expected functionality. After data scientists tune the model, deploying it into production is easy with H<sub>2</sub>O's [POJO/MOJO functionality](https://h2o-release.s3.amazonaws.com/h2o/rel-ueno/2/docs-website/h2o-docs/pojo-quick-start.html). This way, the model is simply exported into a self-containing package, making it easy for the engineers to plug it into any Java-based production environment. Model's performance can be observed even in production at real time, providing useful reports.

Data scientists and developers are best to visit [H2O documentation](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/index.html).It provides [step-by-step tutorials](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/welcome.html#new-users) for new users, as well as guides for users interested in specifically in [R language](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/welcome.html#r-users) or [Python language](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/welcome.html#python-users). The guide gives and overview of what H2O is capable of. Besides a list of [available algorithms](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/data-science.html), methods of [cross-validation](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/cross-validation.html) or the ability to build [models on top of existing ones](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/checkpointing-models.html), ways of [productionizing](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/productionizing.html) created models quickly are described.

In the documentation, everything from data import, exploration & filtering to deployment into production is described. Running on Hadoop ? No problem. Using Apache Spark ? Sparkling Water is at your service. H2O also integrates well with various [cloud services](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/cloud-integration.html). H2O also offers countless [tutorials on GitHub](https://github.com/h2oai/h2o-3/tree/master/h2o-docs/src/product/tutorials).

Video tutorials, speeches and expert advices are available on [H2O's YouTube channel](https://www.youtube.com/watch?v=JWIooAwO7qQ). There is so much to explore.

H<sub>2</sub>O also holds conferences named [H2O World](http://h2oworld.h2o.ai). Come and visit us, we'll be glad to talk to you.