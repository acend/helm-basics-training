---
title: "Labs"
icon: "ti-panel"
description: "Helm Techlabs"
type : "pages"
weight: 2
---

[Helm](https://github.com/helm/helm) is a [Cloud Native Foundation](https://www.cncf.io/) project to define, install and manage applications in Kubernetes.

![Helm Logo](helm-horizontal-color.png)

### tl;dr

Helm is a Package Manager for Kubernetes
* package multiple K8s resources into a single logical deployment unit
* ... but it's not just a Package Manager

Helm is a Deployment Management for Kubernetes
* do a repeatable deployment
* manage dependencies: reuse and share
* manage multiple configurations
* update, rollback and test application deployments

As a recap for after the labs, you can watch *[Amy Chen](https://twitter.com/TheAmyCode)* and her talk at Kubecon North America in 2017.
{{< youtube vQX5nokoqrQ >}}

### Lab Conventions

In order for you to know what is command line input and what is output, we use the following conventions:

Command line input is prefixed by a dollar sign (`$`), e.g.:

```
$ helm version
```

Whereas output is appended directly beneath the command that is supposed to generate it, e.g.:

```
$ helm version
Client: &version.Version{SemVer:"v2.16.5", GitCommit:"89bd14c1541fa93a09492010030fd3699ca65a97", GitTreeState:"clean"}
```

Whenever you see a colored box like the following, you can click on in to open and get more instructions, information or a solution:

{{< collapse box "click here to open & close the box" success>}}
here you find more instructions, the solution or other information.

click again to close the box.
{{< /collapse >}}

{{< notice tip >}}
Carefully read all the labs with its instructions. We suggest to not just simply copy&paste commands, but type them in order to fully understand what you are doing.
{{< /notice >}}

### Prerequisites

* We assume you have knowledge about Kubernetes and understand the concepts behind [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/), [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Services](https://kubernetes.io/docs/concepts/services-networking/service/), [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) and [Secrets](https://kubernetes.io/docs/concepts/services-networking/service/)
* You should also have `kubectl` installed and a working context to access a Kubernetes cluster. Check the [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) for the most common commands.
* Make sure you have access to the internet from your shell. If required, set appropriate proxy settings in your shell. This is only needed for lab 3.0 to access the Helm hub.


### Helm Overview

Ok, let's start with Helm. 
First, you have to understand the following 3 Helm concepts: **Chart**, **Repository** and **Release**.

A **Chart** is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.

A **Repository** is the place where charts can be collected and shared. It’s like Perl’s CPAN archive or the Fedora Package Database, but for Kubernetes packages.

A **Release** is an instance of a chart running in a Kubernetes cluster. One chart can often be installed many times into the same cluster. And each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.

With these concepts in mind, we can now explain Helm like this:

> Helm installs charts into Kubernetes, creating a new release for each installation. And to find new charts, you can search Helm chart repositories.

![Helm Architecture](architecture.png)
*[Image Source](https://www.slideshare.net/alexLM/helm-application-deployment-management-for-kubernetes)*

### Task: Techlab Setup

Make sure you have access to a Kubernetes cluster and `kubectl` is configured to use the right context. For the following labs we are going to create a namespace. You can choose any name, we suggest using e.g. your username. In the labs we are going to use `[USER]` as a placeholder for your created namespace. 

{{< notice tip >}}
Each time you see a `[USER]` somewhere in a command, replace it with your chosen namespace name.
{{< /notice >}}

```bash
$ kubectl create namespace [USER]
``` 

{{< collapse mobi "Mobi-specific instructions" danger>}}
A namespace was already created for you.
{{< /collapse >}}