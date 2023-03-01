---
title: "6.7 Optional: Publish your chart"
weight: 67
sectionnumber: 6.7
---

As you already learned during the presentation, Helm is a great choice if you want to deploy your app in different configurations or over different environments. In this Lab section we are going to show you how you can instantiate the Chart in two different configurations, first a release for the Develop environment and then another release for a Production environment.
Best practice is to deploy them into different namespaces. But thus we have only one, we deploy them into the same namespace but with different release names.


## Task {{% param sectionnumber %}}.1: Deploy the Chart as develop release


Before we deploy the Chart, make following changes in your `values.yaml` file. Append `-dev` to the host names. This makes us later easier to dinstinct between the two releases.

```yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: mychart-<namespace>-dev.labapp.acend.ch
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: mychart-<namespace>-<appdomain>
      hosts:
        - mychart-<namespace>-dev.labapp.acend.ch
```
After that you can install the Chart. Make sure to postfix the release name also with `-dev`.

```bash
helm upgrade -i myapp-dev  ./mychart --namespace <namespace>
```

Next check if the deployment was successfully with the follwing Helm command

```bash
helm ls --namespace <namespace>
```

You should see follwoing output:

//TODO


## Task {{% param sectionnumber %}}.2: Prepare a production release

Now let's create another release for the production environment.
Compared to the develop release we want to adjust following things for our production release:

* Change ingress host name to `mychart-<namespace>-prod.labapp.acend.ch`
* Set the `resource.requests.cpu` to 100m
* Set the `resource.request.memory` to 128Mi

But instead of changing these values in our `values.yaml` file, we create a new empty file called `values-prod.yaml` next to the existing `values.yaml` file.

{{< highlight YAML "hl_lines=15" >}}
.
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── app-deployment.yaml
│   ├── app-ingress.yaml
│   ├── app-service.yaml
│   ├── mariadb-deployment.yaml
│   ├── mariadb-secret.yaml
│   ├── mariadb-service.yaml
│   └── tests
│       └── test-connection.yaml
├── values-prod.yaml
└── values.yaml
{{< /highlight >}}


```bash
helm upgrade -i myapp-prod  ./mychart --namespace <namespace>
```


## Task {{% param sectionnumber %}}.7 Installing chart from repository

When your newly created repository is successfully registered you can install the published chart with the following command:

```bash
helm upgrade -i myapp-from-repo mycharts/mychart
```


## Task {{% param sectionnumber %}}.8: Clean up

Uninstall the app

```bash
helm uninstall myapp-from-repo
```

The repository can be removed by using

```bash
helm repo remove mycharts
```
