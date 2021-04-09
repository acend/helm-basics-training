---
title: "Labs"
weight: 2
menu:
  main:
    weight: 2
---

[Helm](https://github.com/helm/helm) is a [Cloud Native Foundation](https://www.cncf.io/) project to define, install and manage applications in Kubernetes and any Kubernetes distribution like OpenShift.

It can be used to package multiple Kubernetes resources into a single logical deployment unit.

But it's not just a package manager.

Helm is a deployment management for Kubernetes. It can:

* do a repeatable deployment
* manage dependencies, reuse and share
* manage multiple configurations
* update, rollback and test application deployments


### Prerequisites

* We assume you have knowledge about {{% param distroName %}} and understand the concepts of [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/), [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Services](https://kubernetes.io/docs/concepts/services-networking/service/), [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) and [Secrets](https://kubernetes.io/docs/concepts/services-networking/service/).
* You should have `{{% param cliToolName %}}` installed and a working context to access a {{% param distroName %}} cluster. Check the {{% onlyWhenNot openshift %}}[kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/){{% /onlyWhenNot %}}{{% onlyWhen openshift %}}[oc Cheat Sheet](https://developers.redhat.com/cheat-sheets/red-hat-openshift-container-platform){{% /onlyWhen %}} for the most common commands.
* Make sure you have access to the internet from your shell. If required, set appropriate proxy settings in your shell. This is only needed for lab 2 to access the Artifact Hub.


### Helm Overview

Ok, let's start with Helm.
First, you have to understand the following 3 Helm concepts: **Chart**, **Repository** and **Release**.

A **Chart** is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a {{% param distroName %}} cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.

A **Repository** is the place where charts can be collected and shared. It’s like Perl’s CPAN archive or the Fedora Package Database, but for Kubernetes packages.

A **Release** is an instance of a chart running in a {{% param distroName %}} cluster. One chart can often be installed many times in the same cluster. Each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.

With these concepts in mind, we can now explain Helm like this:

> Helm installs charts into Kubernetes, creating a new release for each installation. To find new charts, you can search Helm chart repositories.


### Task: Training Setup

{{% onlyWhen mobi %}}
Make sure you have access to the Mobiliar `kubedev` Kubernetes cluster and `kubectl` is configured to use the right context.
For these labs, we use the Rancher project with name `helm`.
{{% /onlyWhen %}}

{{% onlyWhenNot mobi %}}

{{% onlyWhen rancher %}}


#### Login to the Lab Cluster

Please make sure you have `{{% param cliToolName %}}` installed on your laptop, if not follow the [instructions](https://kubernetes-basics.training.acend.ch/docs/02/) and install it.

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
If you decide to not use the default kubeconfig location at `~/.kube/config` then let `{{% param cliToolName %}}` know where you put it with the KUBECONFIG environment variable:

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
{{% /onlyWhenNot %}}


#### Create Namespace

For the following labs we are going to create a namespace. You can choose any name, we suggest using e.g. your username.

{{% onlyWhen rancher %}}
{{% alert title="Note" color="primary" %}}
Namespaces created via `kubectl` have to be assigned to the correct Rancher Project in order to be visible in the Rancher web console. Please ask your trainer for this assignment or create the namespace directly within this project on the Rancher web console.
{{% /alert %}}
{{% /onlyWhen %}}

You can create your namespace with:

{{% onlyWhenNot openshift %}}

```bash
kubectl create namespace <namespace>
```

{{% /onlyWhenNot %}}{{% onlyWhen openshift %}}

```bash
oc new-project <namespace>
```

{{% /onlyWhen %}}

{{% alert title="Note" color="primary" %}}
We are going to use `<namespace>` as a placeholder for your created namespace. Each time you see a `<namespace>` somewhere in a command, replace it with your chosen namespace name.
{{% /alert %}}
