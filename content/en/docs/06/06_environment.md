---
title: "6.6 Deploy your Chart with different environment configurations"
weight: 66
sectionnumber: 6.6
---


One of the most typical challenges when deploying an application is the handling of different deployment environments.

A typical use case is to deploy your application into a Development and Production environment. Helm is a great solution for handling this challenge.
In this lab we are going to show you how to prepare your Helm releases for different environments.


## Task {{% param sectionnumber %}}.1 Create a devlopment release


Let us start with a new value file for the Development environment. Create a new file called `values-dev.yaml` and add following content to the file

* Set the number of replicas to `1`
* Change the Ingress configuration to match the following path schema `mychart-develop-<namespace>.ocp-internal.cloudscale.puzzle.ch`
* Because we don't need any persistence for the Development environment, disable the database with `database.enabled: false`


```yaml
replicaCount: 1

ingress:
  enabled: true
  hosts:
    - host: mychart-develop-helm.ocp-internal.cloudscale.puzzle.ch
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []

database:
  enabled: false
```

You can specify the '--values'/'-f' flag multiple times. The priority will be given to the last (right-most) file specified. For example, if both myvalues.yaml and override.yaml contained a key called 'Test', the value set in override.yaml would take precedence.

Now install the development release with the following command. First pass the `values.yaml` file which contains all the default values. Then pass the `values-dev.yaml` as second file argument. So the `values-dev.yaml` will overwrite the default Chart values.

```bash
helm upgrade -i myapp-dev .  -f values.yaml -f values-dev.yaml --namespace <namespace> 
```


## Task {{% param sectionnumber %}}.2 Create a production release


Create for the Production environment e new value files named `values-prod.yaml` and add the following settings to the file


* Due to the higher load in our production environment, we change the number of replicas to `3`
* For production usage we also want to persist the data. To enable the database set `database.enabled` to `true`
* Change the database user under `database.databaseuser` to `acend-prod`
* As a best practice, never use the same password in different environemnts. Change the password under `database.databasename`
* Change the Ingress configuration to match the following path schema `mychart-production-<namespace>.ocp-internal.cloudscale.puzzle.ch`


## Task {{% param sectionnumber %}}.3 Solution

The `values-prod.yaml` should look as follow:

```yaml
replicaCount: 3

ingress:
  enabled: true
  hosts:
    - host: mychart-production-helm.ocp-internal.cloudscale.puzzle.ch
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []

database:
  enabled: true
  databaseuser: acend-prod
  databasename: acenddb-prod
```

Install the production release with the following command. Again pass first the default values with `-f values-yaml` and then the production values with `-d values-prod.yaml`

```bash
helm upgrade -i myapp-prod .  -f values.yaml -f values-prod.yaml --namespace <namespace>
```


## Task {{% param sectionnumber %}}.8: Clean up

Uninstall the app

```bash
helm uninstall myapp-prod --namespace <namespace>
helm uninstall myapp-dev --namespace <namespace>
```
