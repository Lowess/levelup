---
title: 🐌 Getting-Started
geekdocNav: true
geekdocAlign: left
geekdocAnchor: false
tags:
    - minikube
    - kubectl
---

{{< hint type="note" title="Objective">}}
* [x] Setup a Kubernetes control plane using `minikube` on your local machine
* [x] Install `kubectl` and diverse utilities
{{< /hint >}}

{{< toc >}}

# Install Pre-requisite utilities

Let's make sure the list of utilities are present on your system before you move on to the next section

{{< tabs "tabs-pre-req-install" >}}

{{< tab "macOS" >}}
Make sure you have [Homebrew](https://brew.sh/index_fr) installed on your system and execute the following commands:

* [jq](https://stedolan.github.io/jq/) is a lightweight and flexible command-line JSON processor.
* [curl](https://curl.se/) is command line tool and library for transferring data with URLs
{{< highlight "sh" >}}
brew install jq curl
{{< /highlight >}}
{{< /tab >}}

{{< tab "Linux" >}}
Make sure your package manager is up to date (`sudo apt-get update`) and execute the following commands

* [jq](https://stedolan.github.io/jq/) is a lightweight and flexible command-line JSON processor.
* [curl](https://curl.se/) is command line tool and library for transferring data with URLs

{{< highlight "sh" >}}
sudo apt-get install jq curl
{{< /highlight >}}
{{< /tab >}}

{{< /tabs >}}

{{< hint type="caution" >}}
Make sure you have :whale2: [Docker](https://docs.docker.com/engine/install/) 18.09 or higher (20.10 or higher is recommended) **installed and started** before you proceed. Indeed `minikube` will be running with the docker driver.
{{< /hint >}}

# Install Minikube

{{< columns >}}

{{< figure src="https://raw.githubusercontent.com/kubernetes/minikube/master/site/static/images/screenshot.png" width="600" >}}

<--->

[Minikube](https://github.com/kubernetes/minikube) implements a local Kubernetes cluster on macOS, Linux, and Windows. minikube's primary goals are to be the best tool for local Kubernetes application development and to support all Kubernetes features that fit.

{{< /columns >}}

* Pick your system and install `minikube`

{{< tabs "tabs-minikube-install" >}}

{{< tab "macOS" >}}
Download and install the latest version of `minikube` using the following command:
{{< highlight "sh" >}}
brew install minikube
{{< /highlight >}}
{{< /tab >}}

{{< tab "Linux" >}}
Download and install the latest version of `minikube` using the following command:

{{< highlight "sh" >}}
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
{{< /highlight >}}
{{< /tab >}}

{{< /tabs >}}

* Now run `minikube version` to make sure everything is setup properly

{{< highlight "sh" >}}
❯ minikube version
minikube version: v1.26.1
commit: 62e108c3dfdec8029a890ad6d8ef96b6461426dc
{{< /highlight >}}

* Let's now start a single node Kubernetes cluster using `minikube start`

{{< highlight "sh" >}}
❯ minikube start
😄  minikube v1.26.1 on Ubuntu 22.04 (kvm/amd64)
✨  Automatically selected the docker driver
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=7800MB) ...
🐳  Preparing Kubernetes v1.24.3 on Docker 20.10.17 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
{{< /highlight >}}

* Here is what just happened on your laptop as a result of this command:

{{< figure src="/excalidraw/00-minikube-setup.excalidraw.png" >}}

> :tada: Congratulations ! You just configured a single node Kubernetes cluster (including a control plane) running inside a local VM on your laptop. Easy right ?

# Install Kubernetes tooling

This section will help you get a couple of important utilities to ease your journey learning Kubernetes.

## `kubectl` - The Kubernetes command line interface

Kubectl is the primary command line interface used to interract with kubernetes. Appologize in advance for GUI users but this time you will have to get your hands a bit dirty :tongue:. Trust me, you will learn later what this cli can do for you and how powerful it is !

{{< hint type="important" >}}
You must use a kubectl version that is within one minor version difference of your cluster. For example, a v1.24 client can communicate with v1.23, v1.24, and v1.25 control planes. Using the latest compatible version of kubectl helps avoid unforeseen issues.
{{< /hint >}}

To ensure you install the right version of `kubectl` we will extract the version of your Kubernetes cluster running in `minikube`, set an environment variable with this version and finally install the proper `kubectl` version.

{{< tabs "tabs-kubectl-install" >}}

{{< tab "macOS - Intel" >}}
You can safely click on the {{< icon "gdoc_copy" >}} icon and paste this script snippet into your terminal.

* Extract the Kubernetes cluster version using minikube command

```sh
export KUBECTL_VERSION=$(minikube kubectl version -- --output=json | jq -r '.serverVersion | (.major + "." + .minor + ".0")')
echo "Your minikube cluster version is: ${KUBECTL_VERSION}"
```

* Download `kubectl`
```sh
curl -LO "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/darwin/amd64/kubectl"
```

* Make the `kubectl` binary executable and move it to `/usr/local/bin`
```sh
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

* Validate `kubectl` is working properly
```sh
kubectl version
```
{{< expand "Terminal Output" >}}
```sh
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", GitCommit:"4ce5a8954017644c5420bae81d72b09b735c21f0", GitTreeState:"clean", BuildDate:"2022-05-03T13:46:05Z", GoVersion:"go1.18.1", Compiler:"gc", Platform:"darwin/amd64"}
Kustomize Version: v4.5.4
```
{{< /expand >}}
{{< /tab >}}

{{< tab "macOS - Apple Silicon (M1)" >}}
You can safely click on the {{< icon "gdoc_copy" >}} icon and paste this script snippet into your terminal.

* Extract the Kubernetes cluster version using minikube command

```sh
export KUBECTL_VERSION=$(minikube kubectl version -- --output=json | jq -r '.serverVersion | (.major + "." + .minor + ".0")')
echo "Your minikube cluster version is: ${KUBECTL_VERSION}"
```

* Download `kubectl`
```sh
curl -LO "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/darwin/arm64/kubectl"
```

* Make the `kubectl` binary executable and move it to `/usr/local/bin`
```sh
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

* Validate `kubectl` is working properly
```sh
kubectl version
```
{{< expand "Terminal Output" >}}
```sh
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", GitCommit:"4ce5a8954017644c5420bae81d72b09b735c21f0", GitTreeState:"clean", BuildDate:"2022-05-03T13:46:05Z", GoVersion:"go1.18.1", Compiler:"gc", Platform:"darwin/arm64"}
Kustomize Version: v4.5.4
```
{{< /expand >}}
{{< /tab >}}

{{< tab "Linux" >}}

You can safely click on the {{< icon "gdoc_copy" >}} icon and paste this script snippet into your terminal.

* Extract the Kubernetes cluster version using minikube command

```sh
export KUBECTL_VERSION=$(minikube kubectl version -- --output=json | jq -r '.serverVersion | (.major + "." + .minor + ".0")')
echo "Your minikube cluster version is: ${KUBECTL_VERSION}"
```

* Download `kubectl`
```sh
curl -LO "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/amd64/kubectl"
```

* Make the `kubectl` binary executable and move it to `/usr/local/bin`
```sh
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
```

* Validate `kubectl` is working properly
```sh
kubectl version
```
{{< expand "Terminal Output" >}}
```sh
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.0", GitCommit:"4ce5a8954017644c5420bae81d72b09b735c21f0", GitTreeState:"clean", BuildDate:"2022-05-03T13:46:05Z", GoVersion:"go1.18.1", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.4
```
{{< /expand >}}
{{< /tab >}}

{{< /tabs >}}

## `kubectx` & `kubens` - Kubernetes companions to make context switching easier

`kubectx` and `kubens` are additional utilities that will make your life **much easier** and prevent you from making mistakes while configuring your `kubectl` profile and switching context to point at the right Kubernetes cluster and the right namespace.

> I won't explain why those utilities are a **must have** but if you want to [learn the hard way](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) go for it !

{{< tabs "tabs-kubectx-kubens-install" >}}

{{< tab "macOS" >}}
Download and install the latest version of `kubectx` and `kubens` using the following commands:
{{< highlight "sh" >}}
brew install kubectx
brew install kubens
{{< /highlight >}}
{{< /tab >}}

{{< tab "Linux" >}}
Download and install the latest version of `kubectx` and `kubens` using the following command:

{{< highlight "sh" >}}
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
{{< /highlight >}}
{{< /tab >}}

{{< /tabs >}}

* Verify both utilities are working properly by running `kubectx` and `kubens` in your terminal, the output should like this

{{< highlight "sh" >}}
❯ kubectx
minikube

❯ kubens
default
kube-node-lease
kube-public
kube-system
{{< /highlight >}}

## `k9s` - (optional) Kubernetes cli to manage your clusters in style!

No GUI does not mean no beauty ! K9s is a nice terminal utility that helps you get an overview of what is running in your cluster. You might find it handy so if you want to install it, feel free to do so.

{{< asciinema id="305944" rows="20" >}}

{{< tabs "tabs-k9s-install" >}}

{{< tab "macOS" >}}
Download and install the latest version of `k9s` using the following command:
{{< highlight "sh" >}}
brew install k9s
{{< /highlight >}}
{{< /tab >}}

{{< tab "Linux" >}}
Download and install the latest version of `k9s` using the following command:

{{< highlight "sh" >}}
curl -sS https://webinstall.dev/k9s | bash
source ~/.config/envman/PATH.env
{{< /highlight >}}
{{< /tab >}}

{{< /tabs >}}

# Before you move on !

Here are a few more `minikube` commands to ensure you don't get stuck if you come back at a later time on this tutorial.

If you shutdown your machine or stop docker, minikube will also stop your local kubernetes cluster.

Here are a few handful commands you should be aware of:

| Action | Commands | Description |
| :--- | :------ | :---------- |
| Cluster lifecycle | `minikube status` / `minikube start` / `minikube delete`  | Manager lifecycle of your minikube instance |
| Dashboard access | `minikube dashboard` | Access the [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) |
| Get IP | `minikube ip` | Retrieves the IP address of the specified node |

And as always, feel free to explore on your all the minikube commands available:

```sh
minikube help

minikube provisions and manages local Kubernetes clusters optimized for development workflows.

Basic Commands:
  start            Starts a local Kubernetes cluster
  status           Gets the status of a local Kubernetes cluster
  stop             Stops a running local Kubernetes cluster
  delete           Deletes a local Kubernetes cluster
  dashboard        Access the Kubernetes dashboard running within the minikube cluster
  pause            pause Kubernetes
  unpause          unpause Kubernetes

Images Commands:
  docker-env       Provides instructions to point your terminal's docker-cli to the Docker Engine inside minikube.
(Useful for building docker images directly inside minikube)
  podman-env       Configure environment to use minikube's Podman service
  cache            Manage cache for images
  image            Manage images

Configuration and Management Commands:
  addons           Enable or disable a minikube addon
  config           Modify persistent configuration values
  profile          Get or list the current profiles (clusters)
  update-context   Update kubeconfig in case of an IP or port change

Networking and Connectivity Commands:
  service          Returns a URL to connect to a service
  tunnel           Connect to LoadBalancer services

Advanced Commands:
  mount            Mounts the specified directory into minikube
  ssh              Log into the minikube environment (for debugging)
  kubectl          Run a kubectl binary matching the cluster version
  node             Add, remove, or list additional nodes
  cp               Copy the specified file into minikube

Troubleshooting Commands:
  ssh-key          Retrieve the ssh identity key path of the specified node
  ssh-host         Retrieve the ssh host key of the specified node
  ip               Retrieves the IP address of the specified node
  logs             Returns logs to debug a local Kubernetes cluster
  update-check     Print current and latest version number
  version          Print the version of minikube
  options          Show a list of global command-line options (applies to all commands).

Other Commands:
  completion       Generate command completion for a shell

Use "minikube <command> --help" for more information about a given command.
```