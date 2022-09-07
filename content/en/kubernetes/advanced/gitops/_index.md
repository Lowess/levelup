---
title: GitOps
geekdocNav: true
geekdocAlign: left
geekdocAnchor: false
tags:
    - helm
---

{{< hint type="note" title="Objective">}}
* [x] Learn what Gitops is about
{{< /hint >}}

{{< toc >}}

{{< hint type="tip" >}}
You might want to also go over this Medium blog post [Streamlining your Kubernetes adoption with Helmfile / ArgoCD and GitOps](https://medium.com/gumgum-tech/streamlining-your-kubernetes-adoption-with-helmfile-argocd-and-gitops-211937e21e29) that recaps the GumGum's Verity™ journey onboarding on GitOps.
{{< /hint >}}


## {{< icon "gdoc_code" >}} What is GitOps ?

> GitOps is a way to do Kubernetes cluster management and application delivery. It works by using Git as a single source of truth for declarative infrastructure and applications.

With GitOps, the use of software agents can alert on any divergence between Git with what’s running in a cluster, and if there’s a difference, Kubernetes reconcilers automatically update or rollback the cluster depending on the case. With Git at the center of your delivery pipelines, developers use familiar tools to make pull requests to accelerate and simplify both application deployments and operations tasks to Kubernetes.

Here are the 4 main principles of a `GitOps` workflow:

{{< columns >}}
# :one:
*The entire system is described declaratively*
<--->
# :two:
*The canonical desired system state is versioned in git*
<--->
# :three:
*Approved changes that can be automatically applied to the system*
<--->
# :four:
*Software agents to ensure correctness and alert on divergence*
{{< /columns >}}


> Read more about GitOps on the [Weaveworks blogpost](https://www.weave.works/technologies/gitops/)

{{< mermaid class="text-center">}}
graph LR;
  dev[Dev] -->|Push| git>App Git Repository]
  git -->|Read only| ci[CI/CD]
  ci -->|Write| registry((Container Registry))
  ci -->|Write| gitconfig>Git Config Repository]
  registry -->|Read only| k8s[Kubernetes Cluster]
  gitconfig -->|Read only| k8s
{{< /mermaid >}}

## {{< icon "gdoc_search" >}} GitOps in practice

{{< slides "https://slides.com/floriandambrine/gitops/" >}}

### :one: The entire system is described declaratively

#### {{< icon "wrench" >}} Software and tooling

The same way you can capture any piece of infrastructure as code using automation tools like [Terraform](https://www.terraform.io/) / [Terragrunt](https://terragrunt.gruntwork.io/), Kubernetes applications deployed to production clusters should be captured as code allowing fast and reliable recovery from scratch.

Kubernetes deployments are usually packaged as [Helm charts](https://helm.sh/docs/topics/charts/). Upon deployment, the chart will be hydrated with values provided by the chart user to configure the given software (what docker image should it run ? How many replicas ? What passwords and secrets should be used ?)

[Helmfile](https://github.com/roboll/helmfile) adds additional functionality to Helm by wrapping it in a declarative spec that allows you to compose several charts together to create a comprehensive deployment artifact for anything from a single application to your entire infrastructure stack.

In addition to the templating and packaging Helm gives you for your Kubernetes manifests, Helmfile provides a way to apply GitOps style CI/CD methodologies over your Helm charts by:

* Separating out your Environment specific information from your Chart
* Performing a diff of your existing deployment and only applying the changes

Helmfile uses the same templating system as Helm and in a way lets you template your templates. This can be a bit difficult to wrap your mind around at first, but adds a ton of powerful features as it allows you to put basic programming logic like if/then/else into just about any component including your actual Helm Chart Values.

Here is an analogy to help you better differentiate them:

| Devops Landscape   | K8s Landscape | Role                                                                                     |
| ------------------ | ------------- | ---------------------------------------------------------------------------------------- |
| `terraform` module | `helm` chart  | Defines what a piece of infrastructure or application should be                          |
| `terragrunt`       | `helmfile`    | Orchestrate multiple pieces of infrastructure or applications to build a wider ecosystem |

#### {{< icon "book" >}} Repository layout

As seen before, the main repository hosting application charts and values will be an `helmfile` repository.

This repository holds the configuration of **every single Kubernetes releases** (Including admin devops components like logging / monitoring / secret management) that will be deployed on clusters.

{{< highlight "sh" >}}
kubernetes-apps-ops
├── .drone.yml
├── Makefile
├── README.md
├── common.yaml
├── environments.yaml
├── helmfile.yaml
└── releases
    ├── gitops ############ Kubernetes User owned releases
    │   ├── helmfile.yaml
    │   └── demoapp
    ├── ops ############### DevOps owned releases
    │   ├── argo-cd
    │   ├── helmfile.yaml
    │   ├── kube-fluentd-operator
    │   ├── kube-prometheus-stack
    │   ├── metrics-server
    │   ├── sealed-secrets
    │   └── traefik
    └── templates ######### Templates shared across charts
        ├── grafana-configmap.gotmpl
        └── sealed-secrets.gotmpl
{{< /highlight >}}

### :two: The canonical desired system state versioned in git

#### New application deployment

1. Define the helm release along with its values for `staging` and `production` environments
2. CI/CD renders helm templates with proper values based on `master` merge (staging) or `git tag` (production)
3. CI/CD pushes rendered templates (`canonical desired state`) into a GitOps repository
4. The Kubernetes cluster uses the GitOps repository as `source of truth` to sync the cluster state

{{< mermaid class="text-center">}}
graph TD;
  subgraph Gitops
    dev[Dev] -->|Push| git>kubernetes-apps-ops]
    git -->|Read only| ci[CI/CD]
    ci -->|1. Helmfile template| cds>Canonical desired state]
    cds -->|2. Rsync| gitconfig>Gitops Config Repository]
    ci -->|3. Push| gitconfig
    gitconfig -->|Read only| k8s[Kubernetes Cluster]
  end
{{< /mermaid >}}


#### Automatic semantic versioning update

{{< hint type="tip" >}}
How do I automatically bump up the version of a docker image on software releases ?
{{< /hint >}}

The idea here is to automatically manage image semantic versioning so that developers do not have to deal with extra steps while integrating with the GitOps workflow. The CI/CD build will be in charge of keeping in sync the project docker image version.

{{< mermaid class="text-center" >}}
graph TD;
  subgraph Auto semver
    dev[Dev] -->|Push| gitapp>myapp]
    gitapp -->|Read only| ci[CI/CD]
    ci -->|1. Publish| reg((Container Image Registry))
    ci -->|2. Auto semver| git>kubernetes-apps-ops]
    ci -->|3. Push or Tag| git
  end
  subgraph Gitops
    git -->|Read only| ci2[CI/CD]
    ci2[CI/CD] -->|1. Helmfile template| cds>Canonical desired state]
    cds -->|2. Rsync| gitconfig>Gitops Config Repository]
    ci2 -->|3. Push| gitconfig
    gitconfig -->|Read only| k8s[Kubernetes Cluster]
  end
{{< /mermaid >}}


Here is Drone CI/CD pipeline to achieve automatic semantic versioning.

{{< highlight "yaml" >}}
---

##############################
### CI/CD pipeline goes here
##############################

kind: pipeline
type: docker
clone:
  disable: true
name: cicd
steps: [...]

##############################
### GitOps CI/CD pipeline
##############################

---

when_deploy: &when_deploy
  when:
    branch: master

kind: pipeline
type: docker
clone:
  disable: true
name: gitops
steps:
  - name: gitops_clone
    image: alpine/git
    commands:
      - git clone <K8S-REPO-WITH-HELM-VALUES-FILES> gitops

  - name: gitops_autosemver
    image: lowess/drone-helm-semver
    pull: always
    settings:
      folder: gitops
      release: <HELM-APP-RELEASE-NAME>
      version: ${DRONE_BUILD_NUMBER}
      allow_multiple: true
    <<: *when_prod

  - name: gitops_push
    image: appleboy/drone-git-push
    settings:
      # SCM Auth
      ssh_key:
        from_secret: gitops_ssh_key
      remote: <K8S-REPO-WITH-HELM-VALUES-FILES>
      # Git Settings
      branch: master
      commit: true
      path: gitops
      commit_message: '[GITOPS] :robot: ${DRONE_COMMIT_MESSAGE/\\n/}'
      author_name: 'GitOps'
      author_email: 'noreply@company.com'

depends_on:
  - cicd
{{< /highlight >}}

### :three: Approved changes that can be automatically applied to the system

Given that projects follow PR review practices and require at least one approval to be merged to master, we assume that changes that go in master should deployable.

{{< hint type="info" >}}
Please note that it's possible to automatically push to a release branch and create a PR instead of pushing straight to master from the CI/CD if required
{{< /hint >}}

### :four: Software agents to ensure correctness and alert on divergence

[ArgoCD](https://argo-cd.readthedocs.io/en/stable/) is a declarative, GitOps continuous delivery tool for Kubernetes. It tracks git repositories that contains application `canonical desired state` files and sync them on the cluster.
