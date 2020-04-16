---
title: "1.0 - Install Helm CLI"
weight: 10
---

This guide shows you how to install the Helm CLI. Helm can be installed either from source or from pre-built binary releases.


### Helm v2 vs v3

Helm recently got a big update to v3 which saw some significant changes. However, Helm v2 is still in heavy use. Before downloading any Helm cli client, make sure you get the correct version.

As for one of the big differences: Helm v2 requires a server-side component to be running inside your Kubernetes cluster called **Tiller**. Tiller is the service that actually communicates with the Kubernetes API to manage our Helm packages.

This implies that Tiller:

* will usually need admin privileges: If a user wants to install a chart that contains any cluster-wide element like a ClusterRole or a CustomResourceDefinition, Tiller should be privileged enough to create or delete those resources.
* should be accessible to any authenticated user: Any valid user of the cluster may require access to install a chart.

That leads to a now-obvious security issue: escalation of privileges. Suddenly, users with minimum privileges are able to interact with the cluster as if they were administrators. The problem is bigger if a Kubernetes pod gets compromised: that compromised pod is also able to access the cluster as an administrator. That's indeed disturbing.

With the v3 release, Helm got rid of Tiller.

{{< notice tip >}}
Check out the [Helm Documentation](https://helm.sh/docs/topics/v2_v3_migration/) for more details about changes between v2 and v3.
{{< /notice >}}


#### Mitigating the issues

The official Helm documentation explains a few hints to mitigate these problems. Unfortunately, they all come with their own issues:

* Reducing Tiller permissions may be an obvious option. That doesn't fit our needs though. You’ll probably still want cluster administrators to be able to install charts with cluster-wide components.
* Securing Tiller communication with TLS mutual authentication. Configuring mutual authentication, users not only need to have a valid user on Kubernetes but also need a client certificate signed by a CA that is trusted by Tiller. A compromised pod isn’t able to access Tiller since you should restrict access to the client certificate by default. The problem is that managing access to the certificate is difficult to maintain. Now cluster administrators need to apply rules to allow or deny access for every new user.
* Running a Tiller instance per namespace. This way you can reduce Tiller permissions for certain instances, while leaving others privileged. Again, the downside with this solution is that it's difficult to maintain and now you are wasting resources, having duplicated deployments. However for the purpose of this Techlab that's how we're going to set it up. This will also mean, that you'll have to add the tiller-namespace Parameter to every helm command eg. `helm ls --namespace [USER] --tiller-namespace [USER]` or as an alternative set the following Environment Variable `export TILLER_NAMESPACE=[USER]`


### Install the Helm cli client

{{< tabs >}}
{{< tab-md "Helm 2" >}}

Every [release](https://github.com/helm/helm/releases) of Helm provides binary releases for a variety of OSes. These binary versions can be manually downloaded and installed.

The latest v2 release v2.16.5 can be found at https://github.com/helm/helm/releases/tag/v2.16.5.


#### Task 1

1. Download your desired version (https://get.helm.sh/helm-v2.16.5-linux-amd64.tar.gz)
1. Unpack it (`tar -zxvf helm-v2.16.5-linux-amd64.tar.gz`)
1. Find the helm binary in the unpacked directory and move it to its desired destination (e.g. `mv linux-amd64/helm /usr/local/bin/helm`)
  * The desired destination should be listed in your $PATH environment variable (`echo $PATH`)

**Windows Users:** Please make sure to select the Windows version https://get.helm.sh/helm-v2.16.5-windows-amd64.zip. Put the binary into your working directory or make sure the directory containing the `helm.exe` binary is in your `Path` environment variable.


To verify run the following command and check if `Version` is what you expected:

```bash
$ helm version
Client: &version.Version{SemVer:"v2.16.5", GitCommit:"89bd14c1541fa93a09492010030fd3699ca65a97", GitTreeState:"clean"}
```

In order to do these labs you're going to need Tiller. It's easiest to install it in your own namespace. We'll also need to create a ServiceAccount, a Role and a RoleBinding for Helm to work correctly:

```bash
$ kubectl create sa "tiller-[USER]" --namespace [USER]
$ kubectl create role "tiller-role-[USER]" --namespace [USER] --verb=* --resource=*.,*.apps,*.batch,*.extensions,*.networking.k8s.io
$ kubectl create rolebinding "tiller-rolebinding-[USER]" --namespace [USER] --role="tiller-role-[USER]" --serviceaccount="[USER]:tiller-[USER]"
$ helm init --service-account "tiller-[USER]" --tiller-namespace [USER] --upgrade
```

{{< /tab-md >}}
{{< tab-md "Helm 3" >}}

Every [release](https://github.com/helm/helm/releases) of Helm provides binary releases for a variety of OSes. These binary versions can be manually downloaded and installed.

The latest v3 release v3.1.2 can be found at https://github.com/helm/helm/releases/tag/v3.1.2.


#### Task 1

Install the `helm` cli on your system:

1. Download your desired version (https://get.helm.sh/helm-v3.1.2-linux-amd64.tar.gz)
1. Unpack it (`tar -zxvf helm-v3.1.2-linux-amd64.tar.gz`)
1. Find the helm binary in the unpacked directory and move it to its desired destination (e.g. `mv linux-amd64/helm /usr/local/bin/`)
  * The desired destination should be listed in your $PATH environment variable (`echo $PATH`)

**Windows Users:** Please make sure to select the Windows version https://get.helm.sh/helm-v3.1.2-windows-amd64.zip. Put the binary into your working directory or make sure the directory containing the `helm.exe` binary is in your `Path` environment variable.

To verify run the following command and check if `Version` is what you expected:

```bash
$ helm version
version.BuildInfo{Version:"v3.1.2", GitCommit:"d878d4d45863e42fd5cff6743294a11d28a9abce", GitTreeState:"clean", GoVersion:"go1.13.8"}
```

From here you should be able to run the client and [add the stable repo](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository):

```bash
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

{{< /tab-md >}}
{{</ tabs >}}

{{< notice warning >}}
**Die Mobiliar**: Please have a look at the following special instructions.
{{< /notice >}}

{{< collapse mobi "Mobi-specific instructions" >}}
The Tiller `init` command installs Tiller in your namespace and by default uses the `gcr.io/kubernetes-helm/tiller:v2.16.5` container image. If your Kubernetes nodes cannot directly pull from the `gcr.io` registry, you can overwrite the image by setting the `--tiller-image` parameter. Use `docker-registry.mobicorp.ch/puzzle/k8s/kurs/tiller:v2.16.5` as your Tiller image:

```bash
$ helm init --service-account "tiller-[USER]" --tiller-namespace [USER] --tiller-image docker-registry.mobicorp.ch/puzzle/k8s/kurs/tiller:v2.16.5 --upgrade
```
{{< /collapse >}}
