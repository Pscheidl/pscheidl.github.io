---
title: "H2O on Kubernetes using Helm"
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

Deploying real-world applications using bare YAML files to Kubernetes is a rather complex task, and H2O is no exception. As demonstrated in one of the [previous blog posts](/h2o-3/h2o-kubernetes-support/). Greatly simplified, a cluster of H2O open source machine learning nodes is brought up in the following manner: 

1. A headless service to make initial node discovery and clustering possible,
1. Leader node is elected within the H2O cluster,
1. Only the leader node marks itself as ready to ensure reproducibility.

The above-mentioned steps require lots of special YAML configuration in place. Without the necessary domain knowledge, it is rather hard to modify the example YAML files for own custom needs and use cases, as almost everything is a factor. The JDK used for the Docker image
must be properly able to recognize resource limits (memory, cpu). Resource requests and limits must always be  the same in order to ensure reproducibility.

One basic way to address this complexity is using [Helm](https://helm.sh/). Helm abstracts its users away from the complexity of assembling the final YAML configuration submitted to Kubernetes. It is a free tool, small and lightweight.
Instead of forcing users to understand specific details about the application deployment, users only provide basic important settings (if any), like resources used by the H2O cluster (CPU, memory), or the cluster size (how many nodes). 

# Quick start

## Install Helm
A necessary prerequisite is to have Helm installed. Helm is available for macOs, Windows and Linux. It is always best to use the official [installation guide](https://helm.sh/docs/intro/install/) with the latest and most up to date information on how to install Helm.

To verify Helm is installed correctly, type `helm` into your console. If help is displayed, Helm is correctly installed.

## Add H2O repository

Once Helm, is installed, it only takes three steps to install H2O into a running Kubernetes cluster:

1. Obtain a KUBECONFIG for the K8S cluster H2O will be deployed to and set the `$KUBECONFIG` environment variable,
1. Add H2O repository (`https://charts.h2o.ai`) to local Helm repositories,
1. Install/deploy H2O to Kubernetes

Helm requires Kubeconfig to know which cluster to talk to and how. Obtaining a KUBECONFIG for a Kubernetes cluster is a vendor-specific operation. Depending on your organization's processes, it might also depend on your organization. This is something your IT deparment should be able to help you with. For example, Red Hat's OpenShift provides
a [CLI tool](https://developers.redhat.com/openshift/command-line-tools) that allows users to obtain the KUBECONFIG.

If you don't have a Kubernetes cluster and would simply like to try H2O out, there is a lightweight Kubernetes cluster named [K3S.io](https://k3s.io/), which may be [installed](https://rancher.com/docs/k3s/latest/en/installation/install-options/) easily by using a [single command](https://rancher.com/docs/k3s/latest/en/installation/install-options/). For K3S, the KUBECONFIG location is typically `/etc/rancher/k3s/k3s.yaml` and
the `$KUBECONFIG` environment variable should be set to point to that location. Alternatively, `the --kubeconfig=/etc/rancher/k3s/k3s.yaml` instruction for Helm can be used.

Once KUBECONFIG is present, the remaining steps are two command line instructions each. The following commands will install a very basic H2O cluster into Kubernetes.

```bash
helm repo add h2o https://charts.h2o.ai
helm install h2o-basic-deployment h2o/h2o-3
``` 

The output should look as follows:

```bash
[pavelp@pavel-alienware17r5 ~]$ helm install h2o-basic-deployment h2o/h2o-3
NAME: h2o
LAST DEPLOYED: Sun Oct 11 15:33:06 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
H2O started successfully.
```

It is also possible to test the deployment:

```bash
[pavelp@pavel-alienware17r5 ~]$ helm test h2o-basic-deployment
Pod h2o-basic-deployment-h2o-3-test-connection pending
Pod h2o-basic-deployment-h2o-3-test-connection running
Pod h2o-basic-deployment-h2o-3-test-connection succeeded
NAME: h2o-basic-deployment
LAST DEPLOYED: Sun Oct 11 15:46:58 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE:     h2o-basic-deployment-h2o-3-test-connection
Last Started:   Sun Oct 11 15:47:10 2020
Last Completed: Sun Oct 11 15:47:31 2020
Phase:          Succeeded
NOTES:
H2O started successfully.
```

The test checks the number of H2O nodes in that very cluster spawned is as expected. The default deployment is designed to be minimalistic and not resource consuming, therefore only one pod is spwaned.

# Advanced options

The basic deployment is due to its minimal resource consumption unusable for real-world deployments. There are many configuration options available for the H2O cluster, enabling users to customize the deployment to their needs.

A list of all available values can be shown by using the `helm inspect` command.

```bash
[pavelp@pavel-alienware17r5 ~]$ helm inspect values h2o/h2o-3
image:
  repository: ""
  name: h2oai/h2o-open-source-k8s
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""
  command: []

h2o:
  nodeCount: 1
  memoryPercentage: 50
  lookupTimeout: 180 # Three minutes by default are very generous amount of time for clustering with parallel pod management
  kubernetesApiPort: 8080

nameOverride: ""
fullnameOverride: ""
podAnnotations: { }
podSecurityContext: { }
securityContext: { }

service:
  port: 80

ingress:
  enabled: false
  annotations: { }
    # kubernetes.io/ingress.class: nginx
  # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: [ ]
  tls: [ ]
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

# For reproducibility, H2O must always have the very same resources allocated throughout the whole container lifetime
resources:
  cpu: 1
  memory: 256Mi

nodeSelector: { }
volumes:
volumeMounts:
tolerations: [ ]
affinity: { }
```

The defaults may change with each release. The default values may be overridden in two ways:

1. By using the `--set` option on the CLI, e.g. `helm install h2o . --kubeconfig /etc/rancher/k3s/k3s.yaml --dry-run set h2o.nodecount=3`
1. By creating an arbitrarily named yaml file (e.g. `myvalues.yaml`), with all the configuration inside. The custom values are then passed in the `install` command as an additional `-f` parameter: `helm install -f myvalues.yaml h2o h2o/h2o-3`.

In this guide, the second approach will be used, assuming users may easily translate the yaml file into a series of `--set` command.

## Changing amout of resources used

Giving H2O enough CPUs and memory is crucial, as well as the number of H2O nodes spawned.

```yaml
h2o:
  nodeCount: 3

resources:
  cpu: 24
  memory: 32Gi
```

The `h2o.nodecount` value dictates how many pods with H2O nodes will be created. There is one H2O node in each pod. 
The `resources.cpu` defines the amount of virtual cpus allocated to each pod, this means to each H2O node. Same rules apply to `resources.memory`, this is the amount of memory allocated to each H2O pod.

As a result, the above example, when saved into a file named `myvalues.yaml` and used by Helm to install H2O-3 (`helm install -f myvalues.yaml h2o h2o/h2o-3`), will create an H2O cluster with three nodes, each with 24 CPUs and 32 Gigabytes of memory.

Another interesting setting is the `h2o.memoryPercentage` value. This value determines how many percent of memory available to each H2O container (running inside the pod) will be available to H2O itself. The default value is `50`, therefore by default, only half of
the memory is allocated for H2O. The reason for that behavior is simple. H2O encapsulates some external algorithms (running in separate processes), like XGBoost. If H2O would allocate all the memory for itself, these external algorithms would not be able to work. 
It is the user's responsibility to change this value according to the actual use case. If XGBoost is not used, allocating about `90` percent of memory just for H2O seems to be a good idea. The result with modified memory percentage allocated by H2O will looks as follows:


```yaml
h2o:
  nodeCount: 3
  memoryPercentage: 90

resources:
  cpu: 24
  memory: 32Gi
```

## Connecting to H2O from outside of the K8S cluster

Exposing H2O and connecting to it via a client from the outside of a Kubernetes cluster belong to basic use cases. One way do do that is by using and `Ingress`. Ingress configuration is heavily dependent on the ingress controller used inside each and every Kubernetes cluster.
Therefore, the following example is adapted to [K3S.io](https://k3s.io) cluster. This binds the ingress to all available hosts and H2O will be available at the root of the context path `/` of your cluster. The adress of an ingress may be found by using the `kubectl get ingresses` command in the very same namespace the H2O ingress has been deployed to.

```yaml
ingress:
  enabled: true
  annotations: {
    kubernetes.io/ingress.class: traefik
  }
  hosts:
    - host: ""
      paths: ["/"]
  tls: []
```

# Acknowledgement

H<sub>2</sub>O.ai is open-source and can be found on [GitHub](https://github.com/h2oai/h2o-3). Found a bug ? Head to H<sub>2</sub>O [JIRA](https://www.pavel.cool/images/mojo_import/mojo_import_1.png). Have questions ? H<sub>2</sub>O offers community [Gitter](https://gitter.im/h2oai/h2o-3) and [Slack](https://www.h2o.ai/slack-community/).