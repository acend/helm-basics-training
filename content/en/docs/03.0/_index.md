---
title: "3. Deploy a more complex Application"
weight: 3
---

In this extended lab, we are going to deploy an existing, more complex application with a Helm chart from the Helm Hub.


{{< onlyWhen helm2 >}}

{{% alert title="Tip" color="warning" %}}
Make sure the Tiller Namespace Environment Variable (`export TILLER_NAMESPACE=[NAMESPACE]`) is set to your Namespace or add the `--tiller-namespace [NAMESPACE]` argument to the helm commands
{{% /alert %}}

{{< /onlyWhen >}}

## Helm Hub

Check out [Helm Hub](https://hub.helm.sh/) where you'll find a huge number of different Helm charts. For this lab, we'll use the [WordPress chart by Bitnami](https://hub.helm.sh/charts/bitnami/wordpress), a publishing platform for building blogs and websites.


## WordPress

As this WordPress Helm chart is published in Bitnami's Helm repository, we're first going to add it to our local repo list:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Let's check if that worked:

```bash
helm repo list
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


{{< onlyWhen mobi >}}
The WordPress and MariaDB charts use (at the time of writing) the following container images:

* `docker.io/bitnami/wordpress:5.4.0-debian-10-r6`
* `docker.io/bitnami/mariadb:10.3.22-debian-10-r60 `

As we cannot access these images, we'll have to overwrite these. Add the following to your `values.yaml` file in order to do so:

```yaml
[...]
image:
  registry: docker-registry.mobicorp.ch
  repository: puzzle/helm-techlab/wordpress

mariadb:
  image:
    registry: docker-registry.mobicorp.ch
    repository: puzzle/helm-techlab/mariadb
[...]
```

You have to merge the `mariadb` part with the already defined `mariadb` part from the lab instructions above. So your final `values.yaml` should look like:

```yaml
---

image:
  registry: docker-registry.mobicorp.ch
  repository: puzzle/helm-techlab/wordpress

persistence:
  size: 1Gi
service:
  type: ClusterIP
updateStrategy: 
  type: Recreate

mariadb:
  image:
    registry: docker-registry.mobicorp.ch
    repository: puzzle/helm-techlab/mariadb
  db:
    password: mysuperpassword123
  master:
    persistence:
      size: 1Gi
```


The image tag remains as already defined in the chart orginial `values.yaml` file from the chart.

You can use the following snippet for your ingress configuration if you want to be able to access the WordPress instance after deploying it (although this is not really necessary for this lab).

```yaml
[...]
ingress:
  enabled: true
  hostname: helmtechlab-wordpress-[USER].phoenix.mobicorp.test
[...]
```

And you have to set your `HTTP_PROXY` environment variable in order to access the bitnami helm repository:

```bash
# Linux
export HTTP_PROXY="http://u...:PASSWORD@dirproxy.mobi.ch:80"
export HTTPS_PROXY="http://u...:PASSWORD@dirproxy.mobi.ch:80"

# Windows cmd
setx HTTP_PROXY="http://u...:PASSWORD@dirproxy.mobi.ch:80"
setx HTTPS_PROXY="http://u...:PASSWORD@dirproxy.mobi.ch:80"
setx http_proxy="http://u...:PASSWORD@dirproxy.mobi.ch:80"
setx https_proxy="http://u...:PASSWORD@dirproxy.mobi.ch:80"

# Windows Powershell
$env:HTTP_PROXY="http://u...:PASSWORD@dirproxy.mobi.ch:80"
$env:HTTPS_PROXY="http://u...:PASSWORD@dirproxy.mobi.ch:80"
$env:http_proxy="http://u...:PASSWORD@dirproxy.mobi.ch:80"
$env:https_proxy="http://u...:PASSWORD@dirproxy.mobi.ch:80"
``` 

If you have specials chars in your password, you have to escape them with hexadecimal value according to https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters
{{< /onlyWhen >}}

The `requirements.yaml` file allows us to define dependencies on other Charts. In our Wordpress Chart we use the `requirements.yaml` to add a `mariadb` to store the Wordpress data in.
```yaml
dependencies:
  - name: mariadb
    version: 7.x.x
    repository: https://charts.bitnami.com/bitnami
    condition: mariadb.enabled
    tags:
      - wordpress-database
```
The Best Practices suggest to use version ranges when ever possible, instead of a fixed version.
The suggested default therefore is patch-level version match:

```
version: ~3.5.7 
```
This for example is equivalent to `>= 3.5.7, < 3.6.0`
Check [Semver](https://github.com/Masterminds/semver#checking-version-constraints) for more information about version ranges.

For more details on how to manage **dependencies**, check out the [Helm Dependencies Documentation](https://v2.helm.sh/docs/charts/#chart-dependencies). 

Subcharts are an alternative way to define dependencies within a chart: A chart may contain (inside of its `charts/` directory) another chart upon which it depends. As a result, when installing the chart, it will install all of its dependencies from the `charts/` directory.

We're now going to deploy the application in a specific version (which is not the latest release on purpose):

{{< onlyWhen helm2 >}}
```bash
helm install --name=wordpress -f values.yaml --namespace [NAMESPACE] --version 9.1.3 bitnami/wordpress
```
{{< /onlyWhen >}}

{{< onlyWhen helm3 >}}
```bash
helm install -f values.yaml --namespace [NAMESPACE] --version 9.1.3 wordpress bitnami/wordpress
```
{{< /onlyWhen >}}

Watch for the newly created resources with `helm ls` and `kubectl get deploy,pod,ingress,pvc`:

```bash
helm ls --namespace [NAMESPACE]
```

which gives you:

```bash
NAME     	NAMESPACE      	REVISION	UPDATED                                 	STATUS  	CHART          	APP VERSION
wordpress	[NAMESPACE]        	1       	2020-03-31 13:23:17.213961038 +0200 CEST	deployed	wordpress-9.0.4	5.3.2
```

```bash
kubectl -n [NAMESPACE] get deploy,pod,ingress,pvc
``` 
which gives you:

```bash
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
helm get values wordpress
``` 
which gives you:

```yaml
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


## Upgrade

We are now going to upgrade the application to a newer Helm chart version. You can do this with:

```bash
helm upgrade -f values.yaml --namespace [USER] --version 9.1.4 wordpress bitnami/wordpress
```

And then observe the changes in your WordPress and MariaDB Apps


## Cleanup

```bash
helm delete wordpress
```

## Additional Task

Study the Helm [Best Practices](https://v2.helm.sh/docs/chart_best_practices/#the-chart-best-practices-guide) as an optional and additional Task.
