---
title: "7.3 Backend as a Dependency"
weight: 73
sectionnumber: 7.3
---

In the previous lab we've manually added a database deployment to our awesome application. Instead of creating your own templates and deployments, we can also integrate other helm charts as dependencies. We have seen this already in lab 3 but are now going to go into more detail in this lab.


## Helm Dependencies

Helm dependencies allow you to integrate other helm charts within your helm chart. It helps to reduce code duplication and to centralize certain functionality and to simply use well maintained helm charts in your deployments.

Dependencies are managed in the Chart.yaml, the dependencies itself are stored in the `charts/` directory of your chart.

In lab 3 we had a look at the wordpress chart, which has a mariadb dependency.


```yaml
annotations:
  category: CMS
apiVersion: v2
appVersion: 5.7.0
dependencies:
- condition: mariadb.enabled
  name: mariadb
  repository: https://charts.bitnami.com/bitnami
  version: 9.x.x
- condition: memcached.enabled
  name: memcached
  repository: https://charts.bitnami.com/bitnami
  version: 5.x.x
- name: common
  repository: https://charts.bitnami.com/bitnami
  tags:
  - bitnami-common
  version: 1.x.x
description: Web publishing platform for building blogs and websites.
home: https://github.com/bitnami/charts/tree/master/bitnami/wordpress
icon: https://bitnami.com/assets/stacks/wordpress/img/wordpress-stack-220x234.png
keywords:
- application
- blog
- cms
- http
- php
- web
- wordpress
maintainers:
- email: containers@bitnami.com
  name: Bitnami
name: wordpress
sources:
- https://github.com/bitnami/bitnami-docker-wordpress
- http://www.wordpress.com/
version: 10.7.2
```


### Conditions

Conditions can be used to enable and disable dependencies. For example if your dev environment has a built in memory database and only your test and prod environment use the mariadb from the dependency.

```yaml
dependencies:
- condition: mariadb.enabled
  name: mariadb
  repository: https://charts.bitnami.com/bitnami
  version: 9.x.x
```

Check [this link](https://helm.sh/docs/chart_best_practices/dependencies/#conditions-and-tags) to learn more about the concept of conditions and tags.


### Choosing Helm Chart as Dependencies

When adding dependencies to your helm charts there are a couple of things you should consider before doing so.

* Be aware of the fact to your helm chart is depending on something you might not have under your control.
* Make sure the helm chart you use as dependency is well maintained, updated regularly, app version is very close to the actual version of the deployed application.
* Explore the chart, specially when using third party helm charts
  * general architecture
  * Functionality: Does the chart really implement all the features you need.
  * Security: Check the default images, do they run as root, security context of pod and filesystem, Service accounts
  * Resource Limits and Requests
* Study default values in the values.yaml
* If the chart doesn't match your needs perfectly, consider implementing the functionality yourself.
* Reduce the amount of dependencies.
* Update your dependencies regularly.


## Task {{% param sectionnumber %}}.1: Use the mariadb bitnami chart as dependency

Let's now replace the mariadb backend, we've manually created in the previous lab with a mariadb chart dependency.

Delete the three templates from the previous lab.

* `templates/service-mariadb.yaml`
* `templates/deployment-mariadb.yaml`
* `templates/secret-mariadb.yaml`

Then we need to add the dependency to the `Chart.yaml`


```yaml
apiVersion: v2
name: mychart
description: My awesome app
type: application
version: 0.1.0
appVersion: 1.16.0
dependencies:
- condition: mariadb.enabled
  name: mariadb
  repository: https://charts.bitnami.com/bitnami
  version: 10.x.x
maintainers:
  - name: <name>
    email: <email>
```

Since we already added the bitnami repository in lab 3 (`helm repo add bitnami https://charts.bitnami.com/bitnami`), after we added the dependency to the Chart.yaml, we can simply run

```bash
helm dependency build .
```

This will download the dependent chart to the `charts/` and update the `Chart.lock` file to the latest matching version according to the version defined in the `Chart.yaml`. The Chart.lock contains the exact version of the moment the dependencies were updates, and it's use to recreate that exact version combinations. In order to update the dependency versions of the Chart.lock file you can simply run `helm dependency update`.


Next we have to add the configuration of our mariadb dependency to the `values.yaml`.
Since the name of our dependency in the `Chart.yaml` is `mariadb`, the configuration of the mariadb must be defined under the mariadb root element.
Let's add the following configuration right after the database section from lab 4.2

```yaml
mariadb:
  enabled: true
  auth:
    rootPassword: mysuperrootpassword123
    database: acenddb
    username: acend
    password: mysuperpassword123
  primary:
    persistence:
      enabled: false
{{% onlyWhen openshift %}}
    podSecurityContext:
      enabled: false
    containerSecurityContext:
      enabled: false
{{% /onlyWhen %}}
```

Update your deployment to match the keys in the `values.yaml` to your environment variables defined in the deployment:

```yaml
          env:
          - name: MYSQL_DATABASE_USER
            value: {{ .Values.mariadb.auth.username }}
          - name: MYSQL_DATABASE_PASSWORD
            value: {{ .Values.mariadb.auth.password }}
          - name: MYSQL_DATABASE_ROOT_PASSWORD
            value: {{ .Values.mariadb.auth.rootPassword }}
          - name: MYSQL_DATABASE_NAME
            value: {{ .Values.mariadb.auth.database }}
          - name: MYSQL_URI
            value: mysql://$(MYSQL_DATABASE_USER):$(MYSQL_DATABASE_PASSWORD)@{{ .Release.Name }}-mariadb/$(MYSQL_DATABASE_NAME)
```

After editing the files we can now install the release.

```bash
helm upgrade -i myapp . --namespace <namespace>
```

Verify the installation and check whether the new database was deployed.

{{% alert title="Note" color="primary" %}}
The whole deployment will take a while until both pods are ready and deployed.
{{% /alert %}}

Run your tests again as soon as the deployments are ready:

```bash
helm test myapp --namespace <namespace>
```


## Task {{% param sectionnumber %}}.2: Explore the bitnami mariadb chart

Use the `--dry-run` option or the `template` command to have a look at the new k8s resources introduced by the dependency.
Explore the [chart source code](https://github.com/bitnami/charts/tree/master/bitnami/mariadb) and have a look at alle the possible [configuration options](https://artifacthub.io/packages/helm/bitnami/mariadb).


## Task {{% param sectionnumber %}}.3: Cleanup

If you're happy with the result, clean up your namespace:

```bash
helm uninstall myapp --namespace <namespace>
```

