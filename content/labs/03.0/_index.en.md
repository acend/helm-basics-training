---
title: "3.0 - Deploy a more complex Application"
weight: 30
---

In this extended lab, we are going to deploy an existing application with a Helm chart

### Helm Hub

Check out [Helm Hub](https://hub.helm.sh/), there you find a lot of Helm charts. For this lab, we choose [Wordpress](https://hub.helm.sh/charts/bitnami/wordpress), a publishing platform for building blogs and websites.

### Wordpress

This Wordpress Helm Chart ist published in the bitnami Helm repository. First, add the bitnami repo to your local repo list:

```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo list
NAME           	URL                                              
bitnami         https://charts.bitnami.com/bitnami 

```

Let's check the available configuration for this Helm chart. Normally you find them in the `values.yaml` File inside the repository or described in the charts readme. You can also check them on the Helm Hub page: https://hub.helm.sh/charts/bitnami/wordpress

We are going to override some of the values, for that purpose, create a new `values.yaml` file locally on your workstation with the following content:

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


If you look inside the [requirements.yaml](https://github.com/bitnami/charts/blob/master/bitnami/wordpress/requirements.yaml) file of the Workdpress Chart you see a dependency to the mariadb Helm chart. All the mariadb values are used by this dependent Helm chart and the chart is automaticly deployed when installing Wordpress.

{{% notice warning %}}
**Die Mobiliar**: Have a look at the following special instructions
{{% /notice %}}

The chart in this version uses as the container images for wordpress and mariadb:

* `docker.io/bitnami/wordpress:5.3.2-debian-10-r48`
* `docker.io/bitnami/mariadb:10.3.22-debian-10-r44`

We have to overwrite these. Add the following to your `values.yaml` file:

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

and you can use the following settings for your ingress if you wan't to open the Wordpress instance after deployment.

```yaml
[...]
ingress:
  enabled: true
  hostname: helmtechlab-wordpress-[USER].phoenix.mobicorp.ch
[...]
```

Now deploy the application with (we choose the Helm Chart version 9.0.4 as we wan't to update later)

```
helm install wordpress -f values.yaml --version 9.0.4 bitnami/wordpress
```

Watch the deployed application with `helm ls` and `kubectl get deploy,pod,ingress,pvc` for the newly created Deployments, the Ingress and also the PersistenceVolumeClaim.

```bash
$ helm ls                                                                
NAME     	NAMESPACE      	REVISION	UPDATED                                 	STATUS  	CHART          	APP VERSION
wordpress	helmtechlab-spl	1       	2020-03-31 13:23:17.213961038 +0200 CEST	deployed	wordpress-9.0.4	5.3.2
```

```bash
kubectl get deploy,pod,ingress,pvc
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

As soon as all Deployments are ready (wordpress and mariadb) you can open the application with the URL from your Ingress defined in `values.yaml`.


### Upgrade

We are now going to upgrade the application to a newer Helm Chart version. You can do this with:

```
helm upgrade -f values.yaml --version 9.1.1 wordpress bitnami/wordpress
```

And then observe the changes in your Wordpress and MariaDB Apps

### Cleanup

```
helm uninstall wordpress
```