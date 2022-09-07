---
title: Deployments - Basics
geekdocNav: true
geekdocAlign: left
geekdocAnchor: false
tags:
    - minikube
    - kubectl
---

{{< hint type="note" title="Objective">}}
* [x] What a Kubernetes `Deployment` is
* [x] Run your first Deployment (hello world)
* [x] Learn `kubectl` commands to interact with deployments

{{< /hint >}}

{{< toc >}}

# :dizzy: What is a Kubernetes `Deployment` ?

> A deployment is an higher abstraction object in Kubernetes. It provides declarative updates for `Pods` and `ReplicaSets`.

> A `ReplicaSet`'s purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.

> A deployment is a commonly used object in Kubernetes. You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate (eg. Releasing a new version of your container application). You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

{{< figure title="Deployments are an higher abstraction object in Kubernetes" src="/excalidraw/01-deployment-hello-world.excalidraw.png" >}}


# :rocket: Run your hello-world `deployment`

Let's reuse the previous `hello-world` example but this time, we will turn it into a Deployment

* Let's go through the definition of a Deployment:


{{< columns >}}

{{< include file="static/workshop/getting-started/hello-world/01-deployments.yaml" language="yaml"  options="linenos=table,hl_lines=1-3 7 8 9 12,linenostart=1" >}}

<--->

{{< hint info >}}
See the [official documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) for more details
{{< /hint >}}

| Field        | Description |
| ------------ | ----------- |
| `apiVersion` | Which version of the Kubernetes API you're using to create this object           |
| `kind` | What kind of object you want to create            |
| `metadata` | Data that helps uniquely identify the object, including a name string            |
| `spec` | What state you desire for the object            |
| `spec.replicas` | Defines how many Pod replicas you want             |
| `spec.selector` | Defines how the Deployment finds which Pods to manage |
| `spec.template` | Defines the Pod template (everything available in Pod object spec is allowed here) |

{{< /columns >}}

* Let's save this deployment spec {{< a_blank "Click to download `01-deployments.yaml`" "/workshop/getting-started/hello-world/01-deployments.yaml" >}} or copy paste the content above and save it to a file

* Before you apply this content to your cluster, let's run a `kubectl` command that will watch deployment objects

```sh
kubectl get deployment --watch
```

* In an other terminal window, go ahead and apply previously created manifest:

```sh
kubectl apply -f 01-deployments.yaml
```

{{< expand "Terminal Output">}}
```sh
$ kubectl get deployment --watch

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
hello-world-deployment   0/3     0            0           0s
hello-world-deployment   0/3     0            0           0s
hello-world-deployment   0/3     0            0           0s
hello-world-deployment   0/3     3            0           0s
hello-world-deployment   1/3     3            1           3s
hello-world-deployment   2/3     3            2           4s
hello-world-deployment   3/3     3            3           5s
hello-world-deployment   3/3     3            3           29s
```
{{< /expand >}}

* :thinking_face: What happened here ?

    * You instructed Kubernetes to create a `Deployment` which itself created a `ReplicaSet` to run 3 instances of your nginx-rainbow `Pod`. You observe that Kubernetes is gradually ramping up the number of Pods to meet your expectations.

    * To verify everything mentioned above, let's verify a `ReplicaSet` was created by running the command `kubectl get replicaset` (or `rs` for shorthand). You can also check that you have the appropriate number of pods running (you know how to do this now :sunglasses:). You will notice
{{< hint type="note" >}}
You will notice that both ReplicaSet and Pods object names contain some sort of SHA after their name.
{{< /hint >}}
    * You can also use `kubectl port-forward deployment/hello-world-deployment 8080:80` to create a tunnel to a deployment object and verify the application is up and running.

## Benefits of a Deployment object

* Let's explore the benefits of wrapping a Pod object inside a Deployment object.

    > :point_up: Take the opportunity to observe and run multiple kubectl commands to inspect what is happening on your cluster.

    * :bomb: Pick a random pod and blow it up using `kubectl delete pod <pod-name>`. What happened ?
{{< expand "Reveal the solution" >}}

{{< hint type="tip" >}}
As soon as the Pod you manually killed died, a new one comes up automatically ! This is the `ReplicaSet` controlled by your deployment that ensure the right amount of pods is running according to your `.spec.replicas` setting
{{< /hint >}}

If you are still watching your deployment you will see the event timeline

```sh
$ kubectl get deployment --watch

...
hello-world-deployment   3/3     3            3           3s
hello-world-deployment   2/3     2            2           7m47s
hello-world-deployment   2/3     3            2           7m47s
hello-world-deployment   3/3     3            3           7m49s
```

While checking your pods you will see the age of one pod being newer than other pods.

```sh {linenos=table,hl_lines=[3],linenostart=1}
$ kubectl get pods
NAME                                     READY   STATUS    RESTARTS   AGE
hello-world-deployment-f5f89f666-6pc2g   1/1     Running   0          5s   <--- Freshly restarted by the ReplicaSet
hello-world-deployment-f5f89f666-8x85h   1/1     Running   0          7m52s
hello-world-deployment-f5f89f666-xtpnw   1/1     Running   0          7m52s
```
{{< /expand >}}
    * :arrow_up: Scale out your deployment to 5 pods now. How can you do this ?
{{< expand "Reveal the solution" >}}

Two options here, imperative or declarative

* Imperative way:

```sh
kubectl scale deployment/hello-world-deployment --replicas 5
```

* Declarative way (prefered so that you capture your changes as code):
```diff {linenos=table,linenostart=1}
# 01-deployments.yaml
...
spec:
-  replicas: 3
+  replicas: 5
...
```
```sh
kubectl apply -f 01-deployments.yaml
```
{{< /expand >}}

  * :package: What about now we release a new version and change from `yellow` to `blue` version ? What do you observe ?
  {{< hint type="tip" >}}
  I advise you to to open a couple of terminal windows to run the following commands in one of each:
  * `kubectl get deployment -w`
  * `kubectl get replicaset -w`
  * `kubectl get pod -w`
  {{< /hint >}}

{{< expand "Reveal the solution" >}}

```diff {linenos=table,linenostart=1}
# 01-deployments.ya
...
spec:
  ...
  template:
    ...
    spec:
      containers:
-      - image: lowess/nginx-rainbow:yellow
+      - image: lowess/nginx-rainbow:blue
...
```
```sh
kubectl apply -f 01-deployments.yaml
```

* Here is what happened when you performed the version switch on your deployment

```sh
❯ kubectl get deployment -w
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
hello-world-deployment   5/5     5            5           37m
hello-world-deployment   5/5     5            5           38m
hello-world-deployment   5/5     5            5           38m
hello-world-deployment   5/5     0            5           38m
hello-world-deployment   5/5     0            5           38m
hello-world-deployment   4/5     0            4           38m
hello-world-deployment   4/5     2            4           38m
hello-world-deployment   4/5     3            4           38m
hello-world-deployment   5/5     3            5           38m
hello-world-deployment   6/5     3            6           38m
hello-world-deployment   6/5     4            6           38m
hello-world-deployment   6/5     4            6           38m
hello-world-deployment   4/5     5            4           38m
hello-world-deployment   5/5     5            5           38m
hello-world-deployment   5/5     5            5           38m
hello-world-deployment   4/5     5            4           38m
hello-world-deployment   5/5     5            5           38m
```

* Replica set

```sh
$ kubectl get replicaset
NAME                               DESIRED   CURRENT   READY   AGE
hello-world-deployment-f5f89f666   5         5         5       37m
hello-world-deployment-7cc466b5b6   2         0         0       0s
hello-world-deployment-f5f89f666    4         5         5       38m
hello-world-deployment-7cc466b5b6   3         0         0       0s
hello-world-deployment-f5f89f666    4         5         5       38m
hello-world-deployment-f5f89f666    4         4         4       38m
hello-world-deployment-7cc466b5b6   3         0         0       0s
hello-world-deployment-7cc466b5b6   3         2         0       0s
hello-world-deployment-7cc466b5b6   3         3         0       0s
hello-world-deployment-7cc466b5b6   3         3         1       2s
hello-world-deployment-f5f89f666    3         4         4       38m
hello-world-deployment-7cc466b5b6   3         3         2       2s
hello-world-deployment-7cc466b5b6   4         3         2       3s
hello-world-deployment-f5f89f666    3         4         4       38m
hello-world-deployment-7cc466b5b6   4         3         2       3s
hello-world-deployment-f5f89f666    3         3         3       38m
hello-world-deployment-7cc466b5b6   4         4         3       3s
hello-world-deployment-f5f89f666    1         3         3       38m
hello-world-deployment-7cc466b5b6   5         4         3       3s
hello-world-deployment-7cc466b5b6   5         4         3       3s
hello-world-deployment-f5f89f666    1         3         3       38m
hello-world-deployment-7cc466b5b6   5         5         3       3s
hello-world-deployment-f5f89f666    1         1         1       38m
hello-world-deployment-7cc466b5b6   5         5         4       7s
hello-world-deployment-f5f89f666    0         1         1       38m
hello-world-deployment-f5f89f666    0         1         1       38m
hello-world-deployment-f5f89f666    0         0         0       38m
hello-world-deployment-7cc466b5b6   5         5         5       7s
```

* Pods

```sh
$ kubectl get pods -w
NAME                                     READY   STATUS    RESTARTS   AGE
hello-world-deployment-f5f89f666-6pc2g   1/1     Running   0          30m
hello-world-deployment-f5f89f666-8x85h   1/1     Running   0          37m
hello-world-deployment-f5f89f666-9kjrq   1/1     Running   0          9m28s
hello-world-deployment-f5f89f666-hw6tr   1/1     Running   0          9m28s
hello-world-deployment-f5f89f666-xtpnw   1/1     Running   0          37m
hello-world-deployment-f5f89f666-9kjrq   1/1     Terminating   0          10m
hello-world-deployment-7cc466b5b6-tbntm   0/1     Pending       0          0s
hello-world-deployment-7cc466b5b6-tjbn8   0/1     Pending       0          0s
hello-world-deployment-7cc466b5b6-tbntm   0/1     Pending       0          0s
hello-world-deployment-7cc466b5b6-tjbn8   0/1     Pending       0          0s
hello-world-deployment-7cc466b5b6-tbntm   0/1     ContainerCreating   0          0s
hello-world-deployment-7cc466b5b6-th9zz   0/1     Pending             0          0s
hello-world-deployment-7cc466b5b6-th9zz   0/1     Pending             0          0s
hello-world-deployment-7cc466b5b6-tjbn8   0/1     ContainerCreating   0          0s
hello-world-deployment-7cc466b5b6-th9zz   0/1     ContainerCreating   0          0s
hello-world-deployment-f5f89f666-9kjrq    0/1     Terminating         0          10m
hello-world-deployment-f5f89f666-9kjrq    0/1     Terminating         0          10m
hello-world-deployment-f5f89f666-9kjrq    0/1     Terminating         0          10m
hello-world-deployment-7cc466b5b6-th9zz   1/1     Running             0          2s
hello-world-deployment-7cc466b5b6-tbntm   1/1     Running             0          2s
hello-world-deployment-f5f89f666-hw6tr    1/1     Terminating         0          10m
hello-world-deployment-7cc466b5b6-s9k8f   0/1     Pending             0          0s
hello-world-deployment-7cc466b5b6-tjbn8   1/1     Running             0          3s
hello-world-deployment-7cc466b5b6-s9k8f   0/1     Pending             0          0s
hello-world-deployment-7cc466b5b6-s9k8f   0/1     ContainerCreating   0          0s
hello-world-deployment-f5f89f666-6pc2g    1/1     Terminating         0          30m
hello-world-deployment-f5f89f666-8x85h    1/1     Terminating         0          38m
hello-world-deployment-7cc466b5b6-9hq7b   0/1     Pending             0          0s
hello-world-deployment-7cc466b5b6-9hq7b   0/1     Pending             0          0s
hello-world-deployment-7cc466b5b6-9hq7b   0/1     ContainerCreating   0          1s
hello-world-deployment-f5f89f666-8x85h    0/1     Terminating         0          38m
hello-world-deployment-f5f89f666-8x85h    0/1     Terminating         0          38m
hello-world-deployment-f5f89f666-8x85h    0/1     Terminating         0          38m
hello-world-deployment-f5f89f666-6pc2g    0/1     Terminating         0          30m
hello-world-deployment-f5f89f666-6pc2g    0/1     Terminating         0          30m
hello-world-deployment-f5f89f666-6pc2g    0/1     Terminating         0          30m
hello-world-deployment-f5f89f666-hw6tr    0/1     Terminating         0          10m
hello-world-deployment-f5f89f666-hw6tr    0/1     Terminating         0          10m
hello-world-deployment-f5f89f666-hw6tr    0/1     Terminating         0          10m
hello-world-deployment-7cc466b5b6-9hq7b   1/1     Running             0          4s
hello-world-deployment-f5f89f666-xtpnw    1/1     Terminating         0          38m
hello-world-deployment-7cc466b5b6-s9k8f   1/1     Running             0          4s
hello-world-deployment-f5f89f666-xtpnw    0/1     Terminating         0          38m
hello-world-deployment-f5f89f666-xtpnw    0/1     Terminating         0          38m
hello-world-deployment-f5f89f666-xtpnw    0/1     Terminating         0          38m
```
{{< /expand >}}

