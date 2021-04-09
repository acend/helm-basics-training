---
title: "Labs"
weight: 2
menu:
  main:
    weight: 2
---


The purpose of these labs is to convey Helm basics by providing hands-on tasks for people. [Helm](https://github.com/helm/helm) is a [Cloud Native Foundation](https://www.cncf.io/) project to define, install and manage applications in Kubernetes and any Kubernetes distribution like OpenShift.
 
It can be used to package multiple Kubernetes resources into a single logical deployment unit.

But it's not just a package manager.
 
Helm is a deployment management for Kubernetes. It can:

* do a repeatable deployment
* manage dependencies, reuse and share
* manage multiple configurations
* update, rollback and test application deployments
 
Goals of these labs:

* Help you get started with this modern technology
* Explain to you the basic concepts
* Show you how to deploy your first applications on Kubernetes using Helm


### Prerequisites

* We assume you have knowledge about {{% param distroName %}} and understand the concepts of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/), [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Services](https://kubernetes.io/docs/concepts/services-networking/service/), [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) and [Secrets](https://kubernetes.io/docs/concepts/services-networking/service/).
* Make sure you have access to the internet from your shell. If required, set appropriate proxy settings in your shell. This is only needed for lab 2 to access the Artifact Hub.


### Helm Overview

Ok, let's start with Helm.
First, you have to understand the following 3 Helm concepts: **Chart**, **Repository** and **Release**.

A **Chart** is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a {{% param distroName %}} cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.

A **Repository** is the place where charts can be collected and shared. It’s like Perl’s CPAN archive or the Fedora Package Database, but for Kubernetes packages.

A **Release** is an instance of a chart running in a {{% param distroName %}} cluster. One chart can often be installed many times in the same cluster. Each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.

With these concepts in mind, we can now explain Helm like this:

> Helm installs charts into Kubernetes, creating a new release for each installation. To find new charts, you can search Helm chart repositories.


### Getting started

After this short introduction, make sure you have completed [setup](../setup/]. You're then ready to start [with the first lab](./01/)!
