---
title: "7.3 Composition"
weight: 73
sectionnumber: 7.3
---

Instead of instantiating the Chart two times manually by hand, we can also define a composition of two Charts in a single Chart. In other 2words we create a new Helm Chart and add the Chart we just wrote twice as a dependency to the chart.

In Helm we can make a composition of already existing Charts and own templates and define it as a new Chart. This is a very common use case if we mix self created resources with own templated resources.

How do we achieve this? Maybe first a bit of theoretical input:

We can declare dependencies in the `Chart.yaml` like the following:

```yaml

# Chart.yaml
dependencies:
- name: memcached
  version: "3.2.1"
  repository: "https://another.example.com/charts"
- name: nginx
  version: "1.2.3"
  repository: "file://../dependency_chart/nginx"
- name: helm-basic-chart
  version: "0.1.0"
  alias: consumer

```

In the example above we can see the `dependency` block of the `Chart.yaml`. We define three dependencies in the Chart and add `memcached`, `nginx` and `helm-basic-chart` as a dependency. As you can see the syntax varies a bit, let's check it for a second:

* `name`: The 'name' should be the name of a chart, where that name must match the name in that chart's 'Chart.yaml' file
* `version`: The 'version' field should contain a semantic version or version range
* `repository`: The 'repository' URL should point to a Chart Repository or to a Chart in your local filesystem
* `alias`: The 'alias' of a dependency (this is very handy to import the same Chart twice)

If we add a dependency without the repository defined, Helm will try to find the defined dependency in either your added repositories or the `/charts` directory in your Chart's directory.

So let's get some work done!


### Task {{% param sectionnumber %}}.1 Create a new Chart

Create a new Chart `parent` and empty the templates folder, since we don't need any additional templates in the Chart other than the dependencies.

{{% details title="Solution" %}}

```bash

helm create parent
rm -rf parent/templates/*

```

{{% /details %}}


### Task {{% param sectionnumber %}}.2 Copy the dependecies

Duplicate the Chart we created in the section before. Copy the duplicated Chart from the section before into your new Chart's `/charts` directory.

Your new Chart's structure should look like this:

```shell

$ tree parent/
parent/
├── Chart.yaml
├── charts
│   └── helm-basic-chart
│       ├── Chart.yaml
│       ├── templates
│       │   ├── _helpers.tpl
│       │   ├── deployment.yaml
│       │   ├── ingress.yaml
│       │   ├── service.yaml
│       │   └── tests
│       │       └── test-connection.yaml
│       └── values.yaml
├── templates
└── values.yaml

```


### Task {{% param sectionnumber %}}.3 Add the chart as dependency

Add two new dependencies to your `parent` Chart's dependencies with the aliases `producer` and `consumer`. Since we have the `helm-basic-chart` (or the name you chose in the dependency) in the local `/charts` directory, we don't need to add any `repository` in the dependency definition.

{{% details title="Solution" %}}

```yaml
# Chart.yaml

apiVersion: v2
name: parent
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
dependencies:
- name: helm-basic-chart
  version: "0.1.0"
  alias: producer
- name: helm-basic-chart
  version: "0.1.0"
  alias: consumer

```

{{% /details %}}


### Task {{% param sectionnumber %}}.4 Configure the values

When we add dependencies to your Helm Charts we can configure them in the parent's `values.yaml`. We can override configuration of dependencies in a parent chart by adding configuration in the `values.yaml` prefixed by the dependencies name or alias. For example if we have a dependency `nginx` in your `Chart.yaml` we can override the configuration of the nginx's `url` property like the following:

```yaml

# parent/values.yaml 
nginx:
    url: samplehost

```

It is your task to configure the two dependencies so they will work just as they worked before!

{{% details title="Hint" %}}

In our example we added two dependencies: `consumer` and `producer`. We can define two blocks of configuration in the parent's `Chart.yaml`:

```yaml

# parent/values.yaml

consumer:
  # Consumer's config

producer:
  # Producer's config

```

{{% /details %}}


{{% details title="Solution" %}}

```yaml

consumer:
  host: consumer-<username>.{{% param labAppUrl %}}
  image:
    name: quay.io/puzzle/quarkus-techlab-data-consumer
    tag: latest
  logLevel: INFO
  resources:
    limits:
      cpu: '1'
      memory: 750Mi
    requests:
      cpu: 50m
      memory: 100Mi
  serviceName: consumer
  producerServiceName: producer-helm-basic-chart-producer

producer:
  host: producer-<username>.{{% param labAppUrl %}}
  image:
    name: quay.io/puzzle/quarkus-techlab-data-producer
    tag: latest
  logLevel: INFO
  resources:
    limits:
      cpu: '1'
      memory: 750Mi
    requests:
      cpu: 50m
      memory: 100Mi
  serviceName: producer

```

{{% /details %}}


### Task {{% param sectionnumber %}}.5 Install the release

Install a Helm Chart release `myrelease` and verify if the two services are running correctly!

{{% details title="Solution" %}}

```bash
helm upgrade -i myrelease parent/.
```
{{% /details %}}

### Task {{% param sectionnumber %}}.5 Verify the release

```bash
curl -kL $({{% param cliToolName %}} get ingress <releasename>-consumer --template="{{(index .spec.rules 0).host}}")/data
{"data":0.4145158804475594}
```



### Task {{% param sectionnumber %}}.6 Clean up

Congratulations! You succeeded in the Chapter and the only thing left is to do some cleanup afterwards!

Remove all the installed objects installed during the chapter!

```bash
helm uninstall myrelease
```
