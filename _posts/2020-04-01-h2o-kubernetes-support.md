---
title: "Running H2O cluster on a Kubernetes cluster"
published: true
classes: wide
categories:
  - H2O-3
tags:
  - H2O.ai
  - H2O-3
  - Machine learning
  - AI
---

The [H2O Open Source](https://github.com/h2oai/h2o-3) is an in-memory platform for distributed, scalable machine learning.
A perfect match for deployment on a Kubernetes cluster, the very modern way of deploying, serving & scaling applications.
With the major release `3.30.0.1`, [released](http://h2o-release.s3.amazonaws.com/h2o/rel-zahradnik/1/index.html) in Q1 2020,
 H2O obtained **first class Kubernetes support**.

In this article, it will not only be explained how to **create H2O deployment** on Kubernetes. It also covers the selected
H2O internal mechanisms for the reader's better understanding of H2O's behavior on Kubernetes cluster.

## Terms used

### H2O

**H2O Node:** One instance of JVM with H2O running inside. Typically, there is one H2O instance per logical machine.

**H2O Cluster:** - Multiple H2O nodes working together on a common computation. The computation is evenly distributed
on each H2O node, typically using Map-Reduce.

### Kubernetes
Please note Kubernetes uses similar terminology as the one to be found in the H2O realm. All the descriptions below are taken from
the [Kubernetes concepts](https://kubernetes.io/docs/concepts/) documentation.

**Kubernetes node:** A node is a worker machine in Kubernetes, previously known as a minion. A node may be a VM or physical machine, depending on the cluster.
Each node contains the services necessary to run pods and is managed by the master components.

**Kubernetes cluster:** A set of node machines for running containerized applications.

## H2O's architecture and behavior on a cluster

In order to understand the behavior and limitations of H2O distributed cluster, it is mandatory to understand the basics of H2O design.

### H2O is stateful
Once H2O nodes are started a cluster is formed (H2O Cluster, not a cluster of Kubernetes nodes). As data are loaded into the H2O cluster,
compression is applied internally and the data are evenly distributed across H2O nodes.

![H2O Internal setup](https://www.pavel.cool/images/h2o-k8s/h2o_frame_distributed.png)

Such approach enables H2O to scale and perform do machine learning on big data. For fast computation, the data are **kept in memory**.
Not only the data loaded, but also all the intermediate computations. Unless explicitly saved to a persistent storage. This is key to H2O's speed.
Once data are loaded and distributed across the cluster of H2O nodes, it is currently impossible to change the cluster configuration.
Adding new nodes, changing node size (memory, CPU) is prohibited, as the cluster adapts to the initial configuration, including data
distribution schemas and compression schemas. The ultimate upside to this approach is H2O's speed.

The information above implies H2O cluster is **stateful**. If one H2O node is terminated, the cluster is immediately recognized as unhealthy and
has to be restarted.

This implies H2O nodes **must be treated as stateful** by K8S. In Kubernetes, a set of pods sharing a common state is named a [Stateful set](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/).
A Kubernetes Stateful Set ensures:

1. H2O Nodes are treated as a single unit, being brought up and down gracefully and together.
1. No attemps will be made by K8S healthcheck (if defined) to restart individual H2O nodes in case of an error.
1. Persistent storages and volumes associated with the StatefulSet of H2O Nodes will not be deleted once the cluster is brought down.

**Key Takeaway:** H2O is a stateful application. H2O Nodes are spawned together and die together. Kubernetes tooling for stateless applications
in not applicable to H2O. Databases are deployed on Kubernetes cluster in a very similar way.

### H2O nodes mutual visibility

In order for H2O to cluster up, it requires addresses of other H2O nodes. On a simple local network, internal heuristics take care of the clustering.
On Kubernetes cluster, the situation gets more difficult, as pods with H2O inside are distributed across Kubernetes nodes and the IP addresses are assigned on demand by default.
Some users utilize the possibility to create and distribute a **flatfile** with H2O Node adresses. This is the **wrong** way. On a Kubernetes cluster,
this represents an additional, very complicated and unnecessary step. First, persistent storage has to be allocated and mounted,
which results in resources and time wasted. Then, Kubernetes cluster has to be queried for the IP addresses of the H2O pods created. The flatfile
is afterwards written to the persistent storage, while the H2O Docker containers inside the pods wait for the file to be present to start H2O. A very
complicated procedure indeed.
 
H2O is now **able to discover** other pods with H2O under the same service **automatically** using resources native Kubernetes - 
**environment variables and services**. No container hacking required.

According to the [Kubernetes networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/),
every Pod gets its own IP address. Also, pods on any node can communicate with all pods on all nodes. Therefore, a `StatefulSet` of H2O Nodes
is created, exposed via a headless service. The clustering is afterwards performed automatically by using DNS records created by the headless service.
The name of the headless service is passed to the H2O pod and then passed all the way to the H2O Docker container by defining `H2O_KUBERNETES_SERVICE_DNS` environment variable.
The format usually follows the `<service-name>.<project-name>.svc.cluster.local` pattern. Once this environment variable is present, H2O assumes it is running inside a Kubernetes
cluster and waits for the clustering to be over before the main H2O program is actually started. All done automatically.

![H2O Clustering](https://www.pavel.cool/images/h2o-k8s/h2o-k8s-clustering.png)

**Key takeaway:** H2O is able to cluster itself inside Kubernetes service using tools native to Kubernetes - services and environment variables.
No other tools required.

### Leader node exposure

In order to ensure reproducibility, all requests should be directed towards H2O Leader node. Leader node election is done
after the node discovery process is completed. Therefore, after the clustering is formed and the leader node is known,
only the pod with H2O leader node should be made available. This also makes the service(s) on top of the deployment route
all requests only to the leader node. To achieve that, readiness probe residing on `/kubernetes/isLeaderNode` address is used.
Once the clustering is done, all nodes but the leader node mark themselves as not ready, leaving only the leader node exposed.

**Key takeaway:** To ensure reproducibility, only the leader not should be contacted. Readiness probe ensures only the
leader node is reachable via the corresponding service.

## Running H2O in Kubernetes cluster

In order to spawn H2O cluster inside a Kubernetes cluster, the following list of requirements must be met:

1. A Kubernetes cluster. For local development, [k3s](https://k3s.io/) is a great choice. For easy start, [OpenShift](https://www.openshift.com/) by RedHat is a great choice with their 30 day free trial.
1. Docker image with H2O inside.
1. A Kubernetes deployment definition with a StatefulSet of H2O pods and a headless service.


### Creating the Docker image

A simple Docker container with H2O running on startup is enough. The simplest way to create one is demonstrated in the figure below.

```Dockerfile
FROM ubuntu:latest

ARG H2O_VERSION

RUN apt-get update \
	&& apt-get install default-jdk unzip wget -y

RUN wget http://h2o-release.s3.amazonaws.com/h2o/rel-zahradnik/1/h2o-${H2O_VERSION}.zip \
	&& unzip h2o-${H2O_VERSION}.zip

ENV H2O_VERSION ${H2O_VERSION}
CMD java -jar h2o-${H2O_VERSION}/h2o.jar
```

To build the Docker image, use the `docker build . -t {image-name} --build-arg H2O_VERSION=3.30.0.1`. Make sure to replace the `{image-name}` placeholder
with a meaningful name. For the purpose of this article, the docker image will be named `h2o-k8s`, resulting in
`docker build . -t h2o-k8s --build-arg H2O_VERSION=3.30.0.1`

### Creating the headless service

H2O Pods deployed on Kubernetes cluster require a [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) 
for H2O Node discovery. The headless service, instead of load-balancing incoming requests to the underlying
H2O pods, returns a set of adresses of all the underlying pods. This enables H2O to cluster itself.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: h2o-service
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: h2o-k8s
  ports:
  - protocol: TCP
    port: 54321
```

The `clusterIP: None` defines the service as headless. The `port: 54321` is the default H2O port. Users and client libraries
use this port to talk to the H2O cluster.

The `app: h2o-k8s` setting is of **great importance**, as this is the name of the application with H2O pods inside. The name
has been chosen arbitrarily to correspoind 
Please make sure this setting corresponds to the name of H2O deployment name chosen.

### Creating the H2O deployment

It is **strongly recommended** to run H2O as a [Stateful set](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
on a Kubernetes cluster. Kubernetes assumes all the pods inside the cluster are stateful and does not attempt to restart
the individual pods on failure. Once a job is triggered on an H2O cluster, the cluster is locked and no additional nodes
can be added. Therefore, the cluster has to be restarted as a whole if required - which is a perfect fit for a StatefulSet.
 

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: h2o-stateful-set
  namespace: h2o-statefulset
spec:
  serviceName: h2o-service
  replicas: 3
  selector:
    matchLabels:
      app: h2o-k8s
  template:
    metadata:
      labels:
        app: h2o-k8s
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: h2o-k8s
          image: '<someDockerImageWithH2OInside>'
          resources:
            requests:
              memory: "4Gi"
          ports:
            - containerPort: 54321
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /kubernetes/isLeaderNode
              port: 8081
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 1
          env:
          - name: H2O_KUBERNETES_SERVICE_DNS
            value: h2o-service.h2o-statefulset.svc.cluster.local
          - name: H2O_NODE_LOOKUP_TIMEOUT
            value: '180'
          - name: H2O_NODE_EXPECTED_COUNT
            value: '3'
          - name: H2O_KUBERNETES_API_PORT
            value: '8081'
```
Besides standardized Kubernetes settings, like `replicas: 3` defining the number of pods with H2O instantiated, there are
several settings to pay attention to.

The name of the application `app: h2o-k8s` must correspond to the name expected by the above-defined headless service in order
for the H2O node discovery to work. H2O communicates on port 54321, therefore `containerPort: 54321`must be exposed to
make it possible for the clients to connect.

The readiness probe residing on `/kubernetes/isLeaderNode` makes sure only the leader node is exposed once the cluster is formed
by making all nodes but the leader node not available. Default port for H2O Kubernetes API is 8080. In the example, an optional
environment variable changes the port to `8081`.

**Environment variables:**

1. `H2O_KUBERNETES_SERVICE_DNS` - **[MANDATORY]** Crucial for the clustering to work. The format usually follows the
 `<service-name>.<project-name>.svc.cluster.local` pattern. This setting enables H2O node discovery via DNS.
  It must be modified to match the name of the headless service created. Also, pay attention to the rest of the address
  to match the specifics of your Kubernetes implementation.
1. `H2O_NODE_LOOKUP_TIMEOUT` - **[OPTIONAL]** Node lookup constraint. Time before the node lookup is ended. 
1. `H2O_NODE_EXPECTED_COUNT` - **[OPTIONAL]** Node lookup constraint. Expected number of H2O pods to be discovered.
1. `H2O_KUBERNETES_API_PORT` - **[OPTIONAL]** Port for Kubernetes API checks and probes to listen on. Defaults to 8080.

If none of the optional lookup constraints is specified, a sensible default node lookup timeout will be set - currently
defaults to 3 minutes. If any of the lookup constraints are defined, the H2O node lookup is terminated on whichever 
condition is met first.

### Exposing H2O cluster

Exposing the H2O cluster is a responsibility of the Kubernetes administrator. By default, an
 [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) can be created. Different platforms offer
 different capabilities, e.g. OpenShift offers [Routes](https://docs.openshift.com/container-platform/4.3/networking/routes/route-configuration.html).

### Putting it all together 

The resulting YAML may be put into a single file, for example `h2o.yaml`.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: h2o-stateful-set
  namespace: h2o-statefulset
spec:
  serviceName: h2o-service
  replicas: 3
  selector:
    matchLabels:
      app: h2o-k8s
  template:
    metadata:
      labels:
        app: h2o-k8s
    spec:
      terminationGracePeriodSeconds: 10
      containers:
        - name: h2o-k8s
          image: 'pscheidl/h2o-k8s'
          resources:
            requests:
              memory: "4Gi"
          ports:
            - containerPort: 54321
              protocol: TCP
          env:
          - name: H2O_KUBERNETES_SERVICE_DNS
            value: h2o-service.h2o-statefulset.svc.cluster.local
          - name: H2O_NODE_LOOKUP_TIMEOUT
            value: '180'
          - name: H2O_NODE_EXPECTED_COUNT
            value: '3'
---
apiVersion: v1
kind: Service
metadata:
  name: h2o-service
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: h2o-k8s
  ports:
  - protocol: TCP
    port: 54321
```

The result might be applied locally using `kubectl apply -f h2o.yaml`, or copied into your favorite Kubernetes cluster provider's interface.

# Acknowledgement
A huge thank you belongs to [Nicholas Anderson](https://www.linkedin.com/in/nanders/) from [Discover Financial Services](https://www.discover.com/), as they were the drivers of H2O Kubernetes implementation, providing a lot of real-world use cases and templates.

Remember, H<sub>2</sub>O.ai is open-source and can be found on [GitHub](https://github.com/h2oai/h2o-3). Found a bug ? Head to H<sub>2</sub>O [JIRA](https://www.pavel.cool/images/mojo_import/mojo_import_1.png). Have questions ? H<sub>2</sub>O offers community [Gitter](https://gitter.im/h2oai/h2o-3) and [Slack](https://www.h2o.ai/slack-community/).