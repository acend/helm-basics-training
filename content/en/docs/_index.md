---
title: "Labs"
weight: 2
menu:
  main:
    weight: 2
---

[Helm](https://github.com/helm/helm) is a [Cloud Native Foundation](https://www.cncf.io/) project to define, install and manage applications in Kubernetes.

It can be used to package multiple K8s resources into a single logical deployment unit.

But it's not just a package manager.

Helm is a deployment management for Kubernetes. It can:

* do a repeatable deployment
* manage dependencies, reuse and share
* manage multiple configurations
* update, rollback and test application deployments

A great recap after finishing the labs is *[Amy Chen](https://twitter.com/TheAmyCode)*'s talk at Kubecon North America in 2017.
{{< youtube vQX5nokoqrQ >}}


### Prerequisites

* We assume you have knowledge about Kubernetes and understand the concepts of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/), [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Services](https://kubernetes.io/docs/concepts/services-networking/service/), [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) and [Secrets](https://kubernetes.io/docs/concepts/services-networking/service/).
* You should have `kubectl` installed and a working context to access a Kubernetes cluster. Check the [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) for the most common commands.
* Make sure you have access to the internet from your shell. If required, set appropriate proxy settings in your shell. This is only needed for lab 3 to access the Helm hub.


### Helm Overview

Ok, let's start with Helm.
First, you have to understand the following 3 Helm concepts: **Chart**, **Repository** and **Release**.

A **Chart** is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.

A **Repository** is the place where charts can be collected and shared. It’s like Perl’s CPAN archive or the Fedora Package Database, but for Kubernetes packages.

A **Release** is an instance of a chart running in a Kubernetes cluster. One chart can often be installed many times in the same cluster. Each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.

With these concepts in mind, we can now explain Helm like this:

> Helm installs charts into Kubernetes, creating a new release for each installation. To find new charts, you can search Helm chart repositories.


### Task: Training Setup

{{% onlyWhenNot mobi %}}

{{% onlyWhen rancher %}}


#### Login to the Lab Cluster

Please make sure you have `kubectl` installed on your laptop, if not follow the [instructions](https://kubernetes-basics.training.acend.ch/docs/02/) and install it.

Our Kubernetes cluster of the lab environment runs on [cloudscale.ch](https://cloudscale.ch) (a Swiss IaaS provider) and has been provisioned with [Rancher](https://rancher.com/). You can log in to the cluster with a Rancher user.

{{% alert title="Note" color="primary" %}}
Your teacher will provide you with the credentials to log in.
{{% /alert %}}

Log in to the Rancher web console and choose the desired cluster.

You now see a button at the top right that says **Kubeconfig File**. Click it, scroll down to the bottom and click **Copy to Clipboard**.

![Download kubeconfig File](kubectlconfigfilebutton.png)

The copied kubeconfig now needs to be put into a file. The default location for the kubeconfig file is `~/.kube/config`.

{{% alert title="Note" color="primary" %}}
If you already have a kubeconfig file, you might need to merge the Rancher entries with yours. Or use a dedicated file as described below.
{{% /alert %}}

Put the copied content into a kubeconfig file on your system.
If you decide to not use the default kubeconfig location at `~/.kube/config` then let `kubectl` know where you put it with the KUBECONFIG environment variable:

```
export KUBECONFIG=$KUBECONFIG:~/.kube-techlab/config
```

{{% alert title="Note" color="primary" %}} When using PowerShell on a Windows Computer use the following command. You'll have to replace `<user>` with your actual user

```
$Env:KUBECONFIG = "C:\Users\<user>\.kube-techlab\config"
```

To set the environment variable (`KUBECONFIG` = `C:\Users\<user>\.kube-techlab\config`) permenantly, check the following documentation:

The `PATH` can be set in Windows in the advanced system settings. It depends on the version:

* [Windows 7](http://geekswithblogs.net/renso/archive/2009/10/21/how-to-set-the-windows-path-in-windows-7.aspx)
* [Windows 8](http://www.itechtics.com/customize-windows-environment-variables/)
* [Windows 10](http://techmixx.de/windows-10-umgebungsvariablen-bearbeiten/)

{{% /alert %}}


{{% /onlyWhen %}}


#### Create Namespace

For the following labs we are going to create a namespace. You can choose any name, we suggest using e.g. your username.

You can create your namespace with:

```bash
kubectl create namespace <namespace>
```

{{% onlyWhen rancher %}}
{{% alert title="Note" color="primary" %}}
Namespaces created via `kubectl` have to be assigned to the correct Rancher Project in order to be visible in the Rancher web console. Please ask your teacher for this assignment. Or you can create the Namespace directly within the Rancher web console.
{{% /alert %}}
{{% /onlyWhen %}}

{{% /onlyWhenNot %}}

{{% onlyWhen mobi %}}
Make sure you have access to the Mobiliar `dev` Kubernetes cluster and `kubectl` is configured to use the right context.

We already created a namespace for you. The name of your namespace is equal to your Mobi U-Account `u....` and has been placed in the Project `helm`.
Just to see it again, a namespace in Kubernetes can be created with:

{{% /onlyWhen %}}

In the labs we are going to use `<namespace>` as a placeholder for your namespace.

{{% alert title="Note" color="primary" %}}
Each time you see a `<namespace>` somewhere in a command, replace it with your chosen namespace name.
{{% /alert %}}

