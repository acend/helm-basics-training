---
title: "3.0 - Deploy a more complex Application"
weight: 30
---

In this extended lab, we are going to deploy an existing, more complex application with a Helm chart from the Helm Hub.

{{< notice tip >}}
Make sure the Tiller Namespace Environment Variable (`export TILLER_NAMESPACE=[USER]`) is set to your Namespace or add the `--tiller-namespace [USER]` argument to the helm commands
{{< /notice >}}

### Helm Hub

Check out [Helm Hub](https://hub.helm.sh/) where you'll find a huge number of different Helm charts. For this lab, we'll use the [WordPress chart by Bitnami](https://hub.helm.sh/charts/bitnami/wordpress), a publishing platform for building blogs and websites.


### WordPress

As this WordPress Helm chart is published in Bitnami's Helm repository, we're first going to add it to our local repo list:

```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

Let's check if that worked:

```bash
$ helm repo list
NAME           	URL                                              
bitnami         https://charts.bitnami.com/bitnami 
```

Let's look at the available configuration for this Helm chart. Usually you can find them in the `values.yaml` or in the charts' readme file. You can also check them on its [Helm Hub page](https://hub.helm.sh/charts/bitnami/wordpress).

We are going to override some of the values. For that purpose, create a new `values.yaml` file locally on your workstation with the following content:

```yaml
---
persistence:
  size: 1Gi
service:
  type: ClusterIP
updateStrategy: 
  type: Recreate

mariadb:
  db:
    password: mysuperpassword123
  master:
    persistence:
      size: 1Gi
```

If you look inside the [requirements.yaml](https://github.com/bitnami/charts/blob/master/bitnami/wordpress/requirements.yaml) file of the WordPress chart you see a dependency to the [MariaDB Helm chart](https://github.com/bitnami/charts/tree/master/bitnami/mariadb). All the MariaDB values are used by this dependent Helm chart and the chart is automatically deployed when installing WordPress.

{{< notice warning >}}
**Die Mobiliar**: Have a look at the following special instructions.
{{< /notice >}}

{{< collapse mobi "Mobi-specific instructions" >}}
The WordPress and MariaDB charts use (at the time of writing) the following container images:

* `docker.io/bitnami/wordpress:5.3.2-debian-10-r48`
* `docker.io/bitnami/mariadb:10.3.22-debian-10-r44`

As we cannot access these images, we'll have to overwrite these. Add the following to your `values.yaml` file in order to do so:

```yaml
[...]
image:
  registry: registry.mobicorp.ch
  repository: puzzle/helm-techlab/wordpress

mariadb:
  image:
    registry: registry.mobicorp.ch
    repository: puzzle/helm-techlab/mariadb
[...]
```

The image tag remains as already defined in the chart `values.yaml` file.

You can use the following snippet for your ingress configuration if you want to be able to access the WordPress instance after deploying it (although this is not really necessary for this lab).

```yaml
[...]
ingress:
  enabled: true
  hostname: helmtechlab-wordpress-[USER].phoenix.mobicorp.ch
[...]
```
{{< /collapse >}}

{{< notice tip >}}
For more details on how to manage dependencies, check out the [Helm Dependencies Documentation](https://v2.helm.sh/docs/charts/#chart-dependencies). Subcharts are an alternative way to define dependencies within a chart: `A chart may contain (inside of its charts/ directory) another chart upon which it depends. In this case, installing the chart will install all of its dependencies. In this case, a chart and its dependencies are managed as a collection.`
{{< /notice >}}


We're now going to deploy the application in a specific version (which is not the latest release on purpose):

{{< tabs >}}{{< tab-md "Helm 2" >}}
```bash
$ helm install --name=wordpress -f values.yaml --namespace [USER] --version 9.0.4 bitnami/wordpress
```
{{< /tab-md >}}{{< tab-md "Helm 3" >}}
```bash
$ helm install -f values.yaml --namespace [USER] --version 9.0.4 wordpress bitnami/wordpress
```
{{< /tab-md >}}{{</ tabs >}}

Watch for the newly created resources with `helm ls` and `kubectl get deploy,pod,ingress,pvc`:

```bash
$ helm ls --namespace [USER]                                                            
NAME     	NAMESPACE      	REVISION	UPDATED                                 	STATUS  	CHART          	APP VERSION
wordpress	[USER]        	1       	2020-03-31 13:23:17.213961038 +0200 CEST	deployed	wordpress-9.0.4	5.3.2
```

```bash
$ kubectl -n [USER] get deploy,pod,ingress,pvc
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress   1/1     1            1           2m6s

NAME                             READY   STATUS    RESTARTS   AGE
pod/wordpress-6bf6df9c5d-w4fpx   1/1     Running   0          2m6s
pod/wordpress-mariadb-0          1/1     Running   0          2m6s

NAME                           HOSTS                                          ADDRESS       PORTS   AGE
ingress.extensions/wordpress   helmtechlab-wordpress.k8s-internal.puzzle.ch   10.100.1.10   80      2m6s

NAME                                             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS            AGE
persistentvolumeclaim/data-wordpress-mariadb-0   Bound    pvc-859fe3b4-b598-4f86-b7ed-a3a183f700fd   1Gi        RWO            cloudscale-volume-ssd   2m6s
persistentvolumeclaim/wordpress                  Bound    pvc-83ebf739-0b0e-45a2-936e-e925141a0d35   1Gi        RWO            cloudscale-volume-ssd   2m7s
```

And use `helm get values wordpress` to deploy the values for a given release.

```bash
$ helm get values wordpress
mariadb:
  db:
    password: mysuperpassword123
  master:
    persistence:
      size: 1Gi
persistence:
  size: 1Gi
service:
  type: ClusterIP
updateStrategy:
  type: Recreate

```

As soon as all deployments are ready (`wordpress` and `mariadb`) you can open the application with the URL from your Ingress defined in `values.yaml`.


### Upgrade

We are now going to upgrade the application to a newer Helm chart version. You can do this with:

```bash
$ helm upgrade -f values.yaml --namespace [USER] --version 9.1.1 wordpress bitnami/wordpress
```

And then observe the changes in your WordPress and MariaDB Apps


### Cleanup

```bash
$ helm delete wordpress
```

### Additional Task

Study the Helm [Best Practices](https://v2.helm.sh/docs/chart_best_practices/#the-chart-best-practices-guide) as an optional and additional Task.
