---
title: "1. Installation"
weight: 1
---

This guide shows you how to install the Helm CLI. Helm can be installed either from source or from pre-built binary releases.


### Helm v2 vs. v3

Helm recently got a big update to v3 which saw some significant changes. However, Helm v2 is still in heavy use. Before downloading any Helm cli client, make sure you get the correct version.

As for one of the big differences: Helm v2 requires a server-side component to be running inside your Kubernetes cluster called **Tiller**. Tiller is the service that actually communicates with the Kubernetes API to manage our Helm packages.

This implies that Tiller:

* will usually need admin privileges: If a user wants to install a chart that contains any cluster-wide element like a ClusterRole or a CustomResourceDefinition, Tiller should be privileged enough to create or delete those resources.
* should be accessible to any authenticated user: Any valid user of the cluster may require access to install a chart.

That leads to a now-obvious security issue: escalation of privileges. Suddenly, users with minimum privileges are able to interact with the cluster as if they were administrators. The problem is bigger if a Kubernetes pod gets compromised: that compromised pod is also able to access the cluster as an administrator. That's indeed disturbing.

With the v3 release, Helm got rid of Tiller.

{{% alert title="Note" color="primary" %}}
Check out the [Helm Documentation](https://helm.sh/docs/topics/v2_v3_migration/) for more details about changes between v2 and v3.
{{% /alert %}}


### Mitigating the issues

The official Helm documentation explains a few hints to mitigate these problems. Unfortunately, they all come with their own issues:

* Reducing Tiller permissions may be an obvious option. That doesn't fit our needs though. You’ll probably still want cluster administrators to be able to install charts with cluster-wide components.
* Securing Tiller communication with TLS mutual authentication. Configuring mutual authentication, users not only need to have a valid user on Kubernetes but also need a client certificate signed by a CA that is trusted by Tiller. A compromised pod isn’t able to access Tiller since you should restrict access to the client certificate by default. The problem is that managing access to the certificate is difficult to maintain. Now cluster administrators need to apply rules to allow or deny access for every new user.
* Running a Tiller instance per namespace. This way you can reduce Tiller permissions for certain instances, while leaving others privileged. Again, the downside with this solution is that it's difficult to maintain and now you are wasting resources, having duplicated deployments. However for the purpose of this Techlab that's how we're going to set it up. This will also mean, that you'll have to add the tiller-namespace Parameter to every helm command eg. `helm ls --namespace [USER] --tiller-namespace [USER]` or as an alternative set the following Environment Variable `export TILLER_NAMESPACE=[USER]`

{{< onlyWhen helm2 >}}


## Install the Helm v2 CLI client

Every [release](https://github.com/helm/helm/releases) of Helm provides binary releases for a variety of OSes. These binary versions can be manually downloaded and installed.

In this lab, we use version v2.16.5 of Helm .The release can be found on [Github](https://github.com/helm/helm/releases/tag/v2.16.5).


## Task 1: Download Helm CLI

1. Download your desired version (e.g. for linux: https://get.helm.sh/helm-v2.16.5-linux-amd64.tar.gz)
1. Unpack it (`tar -zxvf helm-v2.16.5-linux-amd64.tar.gz`)
1. Find the helm binary in the unpacked directory and move it to its desired destination (e.g. `mv linux-amd64/helm /usr/local/bin/helm`)
  * The desired destination should be listed in your $PATH environment variable (`echo $PATH`)

{{% alert title="Warning: Windows Users" color="warning" %}}
Please make sure to select the Windows version https://get.helm.sh/helm-v2.16.5-windows-amd64.zip. Put the binary into your working directory or make sure the directory containing the `helm.exe` binary is in your `Path` environment variable.
{{% /alert %}}

To verify run the following command and check if `Version` is what you expected:

```bash
helm version --client
```

The output is similar to this:

```bash
Client: &version.Version{SemVer:"v2.16.5", GitCommit:"89bd14c1541fa93a09492010030fd3699ca65a97", GitTreeState:"clean"}
```

In order to do these labs you're going to need Tiller. It's easiest to install it in your own namespace. A [ServiceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/), a [Role](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole) and a [RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#default-roles-and-role-bindings) is needed for Helm to work correctly.

{{< onlyWhen mobi >}}
First, make sure to set your http proxy environment variables so the stable chart repository can be added to your helm repos:

```bash
export HTTP_PROXY="http://u...:PASSWORD@dirproxy.mobi.ch:80"
export HTTPS_PROXY="http://u...:PASSWORD@dirproxy.mobi.ch:80"
```

{{% alert title="Note" color="primary" %}}
If you have direct access to the internet from your location, the proxy configuration is not required.
{{% /alert %}}

Replace `u...:PASSWORD` with your account details. If you have specials chars in your password, you have to escape them with hexadecimal value according to https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters

The namespace, the service account, the role and the role binding for Helm have already been created for you.

The Tiller `helm init` command installs Tiller in your namespace and by default uses the `gcr.io/kubernetes-helm/tiller:v2.16.5` container image. Your Mobiliar Kubernetes nodes cannot directly pull from the `gcr.io` registry, you have to overwrite the image by setting the `--tiller-image` parameter. Use `docker-registry.mobicorp.ch/puzzle/k8s/kurs/tiller:v2.16.5` as your Tiller image.

Execute in your terminal to install Tiller in your namespace:

```bash
helm init --service-account "tiller-<NAMESPACE>" --tiller-namespace <NAMESPACE> --tiller-image docker-registry.mobicorp.ch/puzzle/k8s/kurs/tiller:v2.16.5 --upgrade
```

Then wait until tiller deployment is ready. You can check the deployment status with:

```bash
kubectl get deploy tiller-deploy --namespace <NAMESPACE>
```

Now Tiller should be ready and you can check the version of Helm and Tiller with

```bash
helm version --tiller-namespace <NAMESPACE>
```

this should give you an output similar to: 

```bash
Client: &version.Version{SemVer:"v2.16.5", GitCommit:"89bd14c1541fa93a09492010030fd3699ca65a97", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.5", GitCommit:"89bd14c1541fa93a09492010030fd3699ca65a97", GitTreeState:"clean"}
```

{{< /onlyWhen >}}
{{< onlyWhenNot mobi >}}
So lets create the serviceaccount, the role and the rolebinding:

```bash
kubectl create sa "tiller-<NAMESPACE>" --namespace <NAMESPACE>
kubectl create role "tiller-role-<NAMESPACE>" --namespace <NAMESPACE> --verb=* --resource=*.,*.apps,*.batch,*.extensions,*.networking.k8s.io
kubectl create rolebinding "tiller-rolebinding-<NAMESPACE>" --namespace <NAMESPACE> --role="tiller-role-<NAMESPACE>" --serviceaccount="<NAMESPACE>:tiller-<NAMESPACE>"

```

You can verify the created resources with:

```bash
kubectl get sa,role,rolebinding --namespace <NAMESPACE>
```
which gives you an output similar to this:
```bash
NAME                     SECRETS   AGE
serviceaccount/tiller-<NAMESPACE>    1         56s

NAME                                         AGE
role.rbac.authorization.k8s.io/tiller-role-<NAMESPACE>   52s

NAME                                                                                             AGE
rolebinding.rbac.authorization.k8s.io/tiller-rolebinding-[USER]                                         6s
```

then initialize helm with:

```bash
helm init --service-account "tiller-<NAMESPACE>" --tiller-namespace <NAMESPACE> --upgrade
```

Wait until tiller deployment is ready. You can check the deployment status with:
```bash
kubectl get deploy tiller --namespace <NAMESPACE>
```

Now tiller should be ready and you can check the version of helm and tiller with

```bash
helm version --tiller-namespace <NAMESPACE>
```
this should give you an output similar to: 


```bash
TODO
```

{{% alert title="Note" color="primary" %}}
You can delete an existing tiller instance with `kubectl delete deployment tiller --namespace <NAMESPACE>`
{{% /alert %}}

{{< /onlyWhenNot >}}
{{< /onlyWhen >}}
{{< onlyWhen helm3 >}}
## Install the Helm v3 CLI client
Every [release](https://github.com/helm/helm/releases) of Helm provides binary releases for a variety of OSes. These binary versions can be manually downloaded and installed.

The latest v3 release v3.1.2 can be found at https://github.com/helm/helm/releases/tag/v3.1.2.


## Task 1

Install the `helm` cli on your system:

1. Download your desired version (https://get.helm.sh/helm-v3.1.2-linux-amd64.tar.gz)
1. Unpack it (`tar -zxvf helm-v3.1.2-linux-amd64.tar.gz`)
1. Find the helm binary in the unpacked directory and move it to its desired destination (e.g. `mv linux-amd64/helm /usr/local/bin/`)
  * The desired destination should be listed in your $PATH environment variable (`echo $PATH`)

{{% alert title="Windows Users" color="warning" %}}
Please make sure to select the Windows version https://get.helm.sh/helm-v3.1.2-windows-amd64.zip. Put the binary into your working directory or make sure the directory containing the `helm.exe` binary is in your `Path` environment variable.
{{% /alert %}}

To verify run the following command and check if `Version` is what you expected:

```bash
helm version
```

The output is similar to this:
 
```bash
version.BuildInfo{Version:"v3.1.2", GitCommit:"d878d4d45863e42fd5cff6743294a11d28a9abce", GitTreeState:"clean", GoVersion:"go1.13.8"}
```

From here you should be able to run the client and [add the stable repo](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository):

```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```
{{< /onlyWhen >}}
