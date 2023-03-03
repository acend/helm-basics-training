---
title: "1. Getting started"
weight: 1
sectionnumber: 1
---

## Task {{% param sectionnumber %}}.1: Web IDE

The first thing we're going to do is to explore our lab environment and get in touch with the different components.

{{% alert title="Note" color="info" %}}You can also use your local installation of the cli tools. Make sure you completed [the setup](../../setup/) before you continue with this lab.{{% /alert %}}

{{% alert title="Note" color="info" %}}The URL and Credentials to the Web IDE will provided by the teacher. Use Chrome for the best experience.{{% /alert %}}


Once you're successfully logged into the web IDE open a new Terminal by hitting `CTRL + SHIFT + ¨` or clicking the Menu button --> Terminal --> new Terminal and check the installed {{% param cliToolName %}}version by executing the following command:

```bash
{{% param cliToolName %}} version
```

The Web IDE Pod consists of the following tools:

* oc
* kubectl
* kustomize
* helm
* kubectx
* kubens
* tekton cli
* odo
* argocd

The files in the home directory under `/home/project` are stored in a persistence volume, so please make sure to store all your persistence data in this directory.


## Task {{% param sectionnumber %}}.2: Login

{{% onlyWhen mobi %}}
Make sure you have access to the Mobiliar `kubedev` Kubernetes cluster and `kubectl` is configured to use the right context.
For these labs, we use the Rancher project with name `helm`.
{{% /onlyWhen %}}

{{% onlyWhenNot mobi %}}
{{% onlyWhen rancher %}}
Please make sure you have `{{% param cliToolName %}}` installed on your laptop, if not follow the [instructions](https://kubernetes-basics.training.acend.ch/docs/02/) and install it.

Our Kubernetes cluster of the lab environment runs on [cloudscale.ch](https://cloudscale.ch) (a Swiss IaaS provider) and has been provisioned with [Rancher](https://rancher.com/). You can log in to the cluster with a Rancher user.

{{% alert title="Note" color="info" %}}
Your teacher will provide you with the credentials to log in.
{{% /alert %}}

Log in to the Rancher web console and choose the desired cluster.

You now see a button at the top right that says **Kubeconfig File**. Click it, scroll down to the bottom and click **Copy to Clipboard**.

![Download kubeconfig File](kubectlconfigfilebutton.png)

The copied kubeconfig now needs to be put into a file. The default location for the kubeconfig file is `~/.kube/config`.

{{% alert title="Note" color="info" %}}
If you already have a kubeconfig file, you might need to merge the Rancher entries with yours. Or use a dedicated file as described below.
{{% /alert %}}

Put the copied content into a kubeconfig file on your system.
If you decide to not use the default kubeconfig location at `~/.kube/config` then let `{{% param cliToolName %}}` know where you put it with the KUBECONFIG environment variable:

```
export KUBECONFIG=$KUBECONFIG:~/.kube-techlab/config
```

{{% alert title="Note" color="info" %}} When using PowerShell on a Windows Computer use the following command. You'll have to replace `<user>` with your actual user

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

* Open a seperate Browser Tab and login to the OpenShift [Webconsole]({{% param webConsoleURL %}}) using your `<username>` and `<password>`
* Open the menu behind your username in the top right corner
* Hit `Copy Login Command`
* Display the Token and copy the command
* Execute the command in your Webshell terminal.
{{% /onlyWhen %}}


## Task {{% param sectionnumber %}}.3: Namespace

For the following labs we are going to create a namespace. You can choose any name, we suggest using e.g. your username.

{{% onlyWhen rancher %}}
{{% alert title="Note" color="info" %}}
Namespaces created via `kubectl` have to be assigned to the correct Rancher Project in order to be visible in the Rancher web console. Please ask your trainer for this assignment or create the namespace directly within this project on the Rancher web console.
{{% /alert %}}
{{% /onlyWhen %}}

You can create your namespace with:

{{% onlyWhenNot openshift %}}

```bash
{{% param cliToolName %}} create namespace <namespace>
```

{{% alert title="Note" color="info" %}}
We are going to use `<namespace>` as a placeholder for your created namespace. Each time you see a `<namespace>` somewhere in a command, replace it with your chosen namespace name.
{{% /alert %}}

{{% /onlyWhenNot %}}
{{% onlyWhen openshift %}}

```bash
oc new-project <namespace>
```

{{% alert title="Note" color="info" %}}
We are going to use `<namespace>` as a placeholder for your created project. Each time you see a `<namespace>` somewhere in a command, replace it with your chosen project name.
{{% /alert %}}

{{% /onlyWhen %}}


## Task {{% param sectionnumber %}}.3: Helm CLI

The Helm [CLI](https://helm.sh/docs/intro/install/) is a self-contained binary written in the Go programming language. The most recent release can be found on the official [GitHub page](https://github.com/helm/helm/releases). In this chapter you will learn how to get help and how to enable the autocompletion feature.


### Task {{% param sectionnumber %}}.1: Getting familiar with the CLI

You can get help for each command of the CLI with the flag `--help`.

```bash
helm --help
```

You will see a list with the available commands and flags. If you prefer to browse the manual in the browser you’ll find it in the [online documentation](https://helm.sh/docs/helm/).

```
Usage:
  helm [command]

Available Commands:
  completion  generate autocompletion scripts for the specified shell
  create      create a new chart with the given name
  dependency  manage a chart\'s dependencies
  env         helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examine a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      display the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information

Flags:
      --debug                       enable verbose output
  -h, --help                        help for helm
      --kube-apiserver string       the address and the port for the Kubernetes API server
      --kube-as-group stringArray   group to impersonate for the operation, this flag can be repeated to specify multiple groups.
      --kube-as-user string         username to impersonate for the operation
      --kube-ca-file string         the certificate authority file for the Kubernetes API server connection
      --kube-context string         name of the kubeconfig context to use
      --kube-token string           bearer token used for authentication
      --kubeconfig string           path to the kubeconfig file
  -n, --namespace string            namespace scope for this request
      --registry-config string      path to the registry config file (default "/home/bbuehlmann/.config/helm/registry.json")
      --repository-cache string     path to the file containing cached repository indexes (default "/home/bbuehlmann/.cache/helm/repository")
      --repository-config string    path to the file containing repository names and URLs (default "/home/bbuehlmann/.config/helm/repositories.yaml")

Use "helm [command] --help" for more information about a command.
```

You can use the `--help` flag on each command and subcommand of the CLI.

```bash
helm install --help
```

This will print out the documentation for the `install command`. Play around and getting familiar with the different commands of the Helm CLI.


### Task {{% param sectionnumber %}}.1: Autocompletion

{{% alert title="Note" color="info" %}}This step is only needed, when you're not working with the Web IDE we've provided. The autocompletion is already installed in the Web IDE{{% /alert %}}

If you are using the Helm CLI on Linux or Mac OS X you can enable the [autocompletion feature](https://helm.sh/docs/helm/helm_completion/). With autocompletion it's even easier to learn the commands, subcommands and their flags. Last but not least it improves the productivity while using Helm.

The autocompletion feature can be enabled for `bash`, `zsh` and `fish`.

The following example enables autocompletion in the current `bash`:

```bash
source <(helm completion bash)
```

After typing `helm` you can autocomplete the commands and subcommands with a double tap the tabulator key. This works even for installed releases on the cluster: A double tab after `helm get all` prints out all installed helm releases.

To install autocompletion permanently for `bash` you can use the following command:

```bash
echo "source <(helm completion bash)" >> ~/.bashrc
source ~/.bashrc
```

This appends the command `source <(helm completion bash)` to the end of file `~/.bashrc` which will be sourced on launch of the `bash`.
