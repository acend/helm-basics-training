---
title: "1.0 - Install Helm CLI"
weight: 10
---

This guide shows how to install the Helm CLI. Helm can be installed either from source, or from pre-built binary releases.

### Helm v3

Every [release](https://github.com/helm/helm/releases) of Helm provides binary releases for a variety of OSes. These binary versions can be manually downloaded and installed.

v3.1.2 https://github.com/helm/helm/releases/tag/v3.1.2

1. Download your desired version
* Unpack it (`tar -zxvf helm-v3.0.0-linux-amd64.tar.gz`)
* Find the helm binary in the unpacked directory, and move it to its desired destination (`mv linux-amd64/helm /usr/local/bin/helm`)

From there, you should be able to run the client and [add the stable repo](https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository): `helm help`.

### Task 1

Install the `helm` cli on your system. To verify run:

```bash
$ helm version            
version.BuildInfo{Version:"v3.1.1", GitCommit:"afe70585407b420d0097d07b21c47dc511525ac8", GitTreeState:"clean", GoVersion:"go1.13.8"}
```


### Helm v2 vs v3

Helm v2 requires a server-side component to be running inside your Kubernetes cluster, the Helm Server or also know as **Tiller**. Tiller is the service that actually communicates with the Kubernetes API to manage our Helm packages.

This implies that Tiller:

* will usually need admin privileges: If a user wants to install a chart that contains any cluster-wide element like a ClusterRole or a CustomResourceDefinition (CRD), Tiller should be privileged enough to create or delete those resources.
* should be accessible to any authenticated user: Any valid user of the cluster may require access to install a chart.

That leads to a now-obvious security issue: escalation of privileges. Suddenly, users with minimum privileges are able to interact with the cluster as if they were administrators. The problem is bigger if a Kubernetes pod gets compromised: that compromised pod is also able to access the cluster as an administrator. That's indeed disturbing.

With the v3 release, Helm got rid of this Tiller part.

{{% notice tip %}}
Check out the [Helm Documentation](https://helm.sh/docs/topics/v2_v3_migration/) for more details about changes between v2 and v3.
{{% /notice %}}



#### Mitigating the issues

The official Helm documentation explains a few hints to mitigate these problems. Unfortunately, they don't directly suit this case:

* Reducing Tiller permissions may be an obvious option. That doesn't fit our needs though. You’ll probably want cluster administrators to be able to install charts with cluster-wide components.
* Securing Tiller endpoint with a TLS certificate. With this certificate, users not only need to have a valid user, but they also need access to Tiller certificate to contact it. A compromised pod isn’t able to access Tiller since you should restrict access to the certificate by default. The problem is that managing access to the certificate is difficult to maintain. Now cluster administrators need to apply rules to allow or deny access for every new user.
* Running a Tiller instance per namespace. This way you can reduce Tiller permissions for certain instances, while leaving others privileged. Again, the downside with this solution is that it's difficult to maintain and now you are wasting resources, having duplicated deployments.

### Helm v2 and Tiller setup


Every [release](https://github.com/helm/helm/releases) of Helm provides binary releases for a variety of OSes. These binary versions can be manually downloaded and installed.

v2.16.5 https://github.com/helm/helm/releases/tag/v2.16.5

1. Download your desired version
* Unpack it (`tar -zxvf helm-v2.16.5-linux-amd64.tar.gz`)
* Find the helm binary in the unpacked directory, and move it to its desired destination (`mv linux-amd64/helm /usr/local/bin/helm`)

For this Exercise, you can install Tiller in your own Namespace, but we also need to create a ServiceAccount and a Role & Rolebinding for Helm to work correctly:

```
kubectl create sa "tiller-[USER]"
kubectl create role "tiller-role-[USER]" --namespace [USER] --verb=* --resource=*.,*.apps,*.batch,*.extensions
kubectl create rolebinding "tiller-rolebinding-[USER]" --role="tiller-role-[USER]" --serviceaccount="[USER]:tiller-[USER]"
helm init --service-account "tiller-[USER]" --tiller-namespace [USER] --upgrade
```

{{% notice warning %}}
**Die Mobiliar:**: Have a look at the following special instructions
{{% /notice %}}

The Tiller init command does install tiller on your Namespace and uses the `gcr.io/kubernetes-helm/tiller:v2.16.5` container image. If your Kubernetes nodes cannot dirrectly pull from the gcr.io registry, you can overwrite the images used by setting the image via `--tiller-image` argument. You can use `docker-registry.mobicorp.ch/puzzle/k8s/kurs/tiller:v2.16.5` as your Tiller image.

```
helm init --service-account "tiller-[USER]" --tiller-namespace [USER] --tiller-image docker-registry.mobicorp.ch/puzzle/k8s/kurs/tiller:v2.16.5 --upgrade
```
