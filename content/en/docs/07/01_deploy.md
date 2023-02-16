---
title: "7.1 Deploy your awesome application"
weight: 71
sectionnumber: 7.1
---

Using the generated and modified Helm chart, we are going to deploy our own awesome application.


## Task {{% param sectionnumber %}}.1: Deploy the chart

Let's us deploy our awesome application. Therefore we need to adjust the values file.

```yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: mychart-<namespace>.labapp.acend.ch
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: mychart-<namespace>-<appdomain>
      hosts:
        - mychart-<namespace>.labapp.acend.ch
```


### Solution



{{% onlyWhenNot mobi %}}
{{% alert title="Note" color="primary" %}}
Don't forget to replace `{{% param labAppUrl %}}` with the value provided by the trainer.
{{% /onlyWhenNot %}}
{{% onlyWhen mobi %}}
It might take some time until your ingress hostname is accessible, as the DNS name first has to be propagated correctly.
{{% /onlyWhen %}}
{{% /alert %}}


To create a release from our chart, we run the following command within our chart directory:

```bash
helm upgrade -i myapp ./mychart --namespace <namespace>
```

This will create a new release with the name `myapp`. If we already had installed a release and wanted to update the existing one, we'd use the following command:

```bash
helm upgrade <existingrelease> --namespace <namespace> ./mychart
```

Check whether the ingress was successfully deployed by accessing the URL `http://mychart-<namespace>.{{% param labAppUrl %}}/`

Continue with the lab "[A new backend](../database/)".
