---
title: Terminology
geekdocNav: true
geekdocAlign: left
geekdocAnchor: false
tags:
    - kubernetes
    - terminology
---

{{< hint type="note" title="Objective">}}
* [x] Learn Kubernetes common terms and get familiar with them
{{< /hint >}}

{{< toc >}}

## Glossary 
### Cluster

> A set of worker machines, called nodes, that run containerized applications. Every cluster has at least one worker node.
{{< expand "more details..." >}}
The worker node(s) host the Pods that are the components of the application workload. The control plane manages the worker nodes and the Pods in the cluster. In production environments, the control plane usually runs across multiple computers and a cluster usually runs multiple nodes, providing fault-tolerance and high availability.
{{< /expand >}}

### Control Plane 

> The container orchestration layer that exposes the API and interfaces to define, deploy, and manage the lifecycle of containers.
{{< expand "more details..." >}}
This layer is composed by many different components, such as (but not restricted to):

* etcd
* API Server
* Scheduler
* Controller Manager
* Cloud Controller Manager
{{< /expand >}}

