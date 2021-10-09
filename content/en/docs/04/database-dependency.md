---
title: "4.3 Backend as a Dependency"
weight: 43
sectionnumber: 4.3
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

Delete the two templates from the previous lab. `templates/service-mariadb.yaml` and `templates/deployment-mariadb.yaml`

Then we need to add the dependency to the `Chart.yaml`


```yaml
apiVersion: v2
name: mychart
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"
dependencies:
- condition: mariadb.enabled
  name: mariadb
  repository: https://charts.bitnami.com/bitnami
  version: 9.x.x
maintainers:
  - name: <name>
    email: <email>
```

Since we already added the bitnami repository in lab 3 (`helm repo add bitnami https://charts.bitnami.com/bitnami`), after we added the dependency to the Chart.yaml, we can simply run

```bash
helm dependency build
```

This will download the dependent chart to the `charts/` and update the `Chart.lock` file to the latest matching version according to the version defined in the `Chart.yaml`. The Chart.lock contains the exact version of the moment the dependencies were updates, and it's use to recreate that exact version combinations. In order to update the dependency versions of the Chart.lock file you can simply run `helm dependency update`.


Next we have to configure the dependency in your `values.yaml`, let's add the following configuration right after the database section from lab 4.2

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
      size: 1Gi
{{% onlyWhen openshift %}}
    podSecurityContext:
      enabled: false
    containerSecurityContext:
      enabled: false
{{% /onlyWhen %}}
```


```bash
helm upgrade myapp ./mychart --namespace <namespace>
```

Verify the installation and check whether the new database was deployed.

