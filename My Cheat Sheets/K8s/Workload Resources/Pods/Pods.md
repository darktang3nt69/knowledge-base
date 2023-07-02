# Pods

_Pods_ are the smallest deployable units of computing that you can create and manage in Kubernetes.

A _Pod_ (as in a pod of whales or pea pod) is a group of one or more [containers](https://kubernetes.io/docs/concepts/containers/), with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. A Pod models an application-specific "logical host": it contains one or more application containers which are relatively tightly coupled. In non-cloud contexts, applications executed on the same physical or virtual machine are analogous to cloud applications executed on the same logical host.

As well as application containers, a Pod can contain [init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) that run during Pod startup. You can also inject [ephemeral containers](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) for debugging if your cluster offers this.

## What is a Pod?[](https://kubernetes.io/docs/concepts/workloads/pods/#what-is-a-pod)

**Note:** While Kubernetes supports more [container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes) than just Docker, [Docker](https://www.docker.com/) is the most commonly known runtime, and it helps to describe Pods using some terminology from Docker.

The shared context of a Pod is a set of Linux namespaces, cgroups, and potentially other facets of isolation - the same things that isolate a [container](https://kubernetes.io/docs/concepts/containers/). Within a Pod's context, the individual applications may have further sub-isolations applied.

A Pod is similar to a set of containers with shared namespaces and shared filesystem volumes