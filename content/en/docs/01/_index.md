---
title: "1. Getting started"
weight: 1
sectionnumber: 1
---

## Task {{% param sectionnumber %}}.1: `{{% param cliToolName %}}` installation

You should have a current version of `{{% param cliToolName %}}` installed on your machine.


## Task {{% param sectionnumber %}}.2: Login

{{% onlyWhen mobi %}}
Make sure you have access to the Mobiliar `kubedev` Kubernetes cluster and `kubectl` is configured to use the right context.
For these labs, we use the Rancher project with name `helm`.
{{% /onlyWhen %}}

{{% onlyWhenNot mobi %}}
{{% onlyWhen rancher %}}
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

{{% onlyWhen openshift %}}
Please refer to the information your teacher provided you with.
{{% /onlyWhen %}}


## Task {{% param sectionnumber %}}.3: Namespace

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
