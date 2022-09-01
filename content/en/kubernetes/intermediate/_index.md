---
title: 🐧 Intermediate
geekdocNav: true
geekdocAlign: left
geekdocAnchor: false
tags:
    - helm
---

{{< hint type="note" title="Objective">}}
* [x] Learn what `helm` is and install it on your local machine
* [x] Use helm to install open-source Kubernetes applications
{{< /hint >}}

{{< toc >}}

# What is `helm` ?

> Helm is a package manager for Kubernetes

In simple terms, Helm is a package manager for Kubernetes. Helm is the K8s equivalent of `yum` or `apt` on the system side or even `npm` or `pip` for programming language equivalent.

Helm deploys `charts`, which you can think of as a packaged application. It is a collection of all the versioned, pre-configured application resources which can be deployed as one unit. You can then deploy another version of the chart with a different set of configuration.

A chart is made of [Go templates](https://helm.sh/docs/chart_template_guide/function_list/) that provides functions to help generate Kubernetes YAML manifest dynamically.

`Helm` helps in three key ways:

* Improves productivity
* Reduces the complexity of deployments of microservices
* Enables the adaptation of cloud native applications


A single `chart` might be used to deploy something simple, like a webserver pod, or something much more complex, like:

* A fully functioning Kafka eco-system made of a [Zookeeper cluster](https://zookeeper.apache.org/), [Kafka brokers](https://kafka.apache.org/), [Kafka Connects](https://docs.confluent.io/platform/current/connect/index.html) and a [Schema-registry](https://docs.confluent.io/platform/current/schema-registry/index.html)

* An application stack with HTTP servers, SQL databases, Redis caches, and so on...

# Install Pre-requisite utilities

TODO