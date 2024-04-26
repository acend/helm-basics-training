---
title: "7.1 Refactor 1"
weight: 71
sectionnumber: 7.1
---

## Task {{% param sectionnumber %}}.1: Make the Ingress resource more configurable

After cloning the repository and inspecting the templates already created for you, you will notice some potential for improvement. For example the hostname of the applications should not be hard-coded. When deploying to multiple environments you will run into conflicts.

Modify the **consumer and producer** ingress templates and extract following variables to make them configurable:

* Extract `.spec.rules.host` as value
* Extract `.spec.tls.hosts[0]` as value, use the same value as above


{{% alert title="Note" color="info" %}}
If you want to test the chart locally you can execute following command

`helm template -s templates/consumer-ingress.yaml ./helm-basic-chart --debug | cat -n -`
{{% /alert %}}

First let us define the new variables in our `values.yaml` file. Replace `<username>` with your username


{{% onlyWhenNot openshift %}}

{{< highlight YAML "hl_lines=2 6" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/main/helm-basic-chart/values.yaml" %}}
{{< /highlight >}}

{{% /onlyWhenNot  %}}
{{% onlyWhen openshift %}}

```yaml
producer:
  host: producer-<namespace>.training.openshift.ch

consumer:
  tag: latest
  host: consumer-<namespace>.training.openshift.ch
```
{{% /onlyWhen  %}}

Next replace the hard coded values for the host value in our `consumer-ingress.yaml` file.

{{< highlight YAML "hl_lines=11 23" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/templates/consumer-ingress.yaml" %}}
{{< /highlight >}}

Replace the same value in our `producer-ingress.yaml` file.

{{< highlight YAML "hl_lines=11 23" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/templates/producer-ingress.yaml" %}}
{{< /highlight >}}


Afterwards we can install our Helm Chart with following command.

```bash
helm upgrade -i myrelease --namespace $USER ./helm-basic-chart
```

Verify your deployment! Check if your pods are running and healthy!

```bash
{{% param cliToolName %}} get pods
```

This should return something like this:

```
{{% param cliToolName %}} get pods
NAME                             READY   STATUS    RESTARTS   AGE
data-consumer-7686976d88-2wbh5   1/1     Running   0          72s
data-producer-786d6bb688-qpg4c   1/1     Running   0          72s
```

Pay attention to the `1/1` status in the ready section! After checking if our pods are ready and therefore will accept traffic from the service, check if the application is running correctly:

Both applications (data-producer and data-consumer) you just deployed expose an endpoint `/data` which will return a random double data json. Try to verify if your deployment is running correctly by using curl.

{{% details title="Hint 1" %}}

If you are struggling with curl you can use the following syntax:

```bash
curl http(s)://HOST
```

When the problem will be a redirect or certificate problem, try the flags `-L` and `-k` to mitigate the error.

{{% /details %}}

{{% details title="Solution" %}}

```bash

curl -kL $({{% param cliToolName %}} get ingress <releasename>-consumer --template="{{(index .spec.rules 0).host}}")/data
{"data":0.15495350024595755}

```

If your application returns the data point when consuming the consumers `/data` endpoint, then both applications work.

{{% /details %}}


## Task {{% param sectionnumber %}}.2: Make the deployments more configurable

Not just the ingresses could use some improvements. Also the deployments could be more configurable. In order to keep up to date with the current requirements your task is to adapt the following things:

Producer Deployment:

* Extract the image tag from the `.spec.containers[0].image` on Line 22 field as value
* Extract the `.spec.containers[0].resources` block from line 51 as value `consumer.resources`. Make use of the `toYaml` and the `nindent` function.
* Extract the `.spec.containers[0].env["QUARKUS_LOG_LEVEL"]` on line 26 block as value

Consumer Deployment:

* Extract the `.spec.containers[0].resources` block from line 51 as value `producer.resources`. Make use of the `toYaml` and the `nindent` function.
* Extract the `.spec.containers[0].env["QUARKUS_LOG_LEVEL"]` on line 26 block as value `producer.logLevel`

{{% alert title="Note" color="info" %}}
Take a look at the official Helm documentation for a list of built in functions.

[Built In Helm functions](https://helm.sh/docs/chart_template_guide/function_list/)
{{% /alert %}}

{{% details title="Solution" %}}


### producer-deployment.yaml

{{< highlight YAML "hl_lines=22 26 51" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/templates/producer-deployment.yaml" %}}
{{< /highlight >}}


### consumer-deployment.yaml

{{< highlight YAML "hl_lines=26 51" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/templates/consumer-deployment.yaml" %}}
{{< /highlight >}}


### values.yaml

{{< highlight YAML "hl_lines=2-11 14-22" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/values.yaml" %}}
{{< /highlight >}}

{{% /details %}}


## Task {{% param sectionnumber %}}.3: Upgrade the chart

Execute following command to update our helm release.

```shell
helm upgrade myrelease --namespace $USER ./helm-basic-chart
```

Finally, you can visit your application with the URL provided from the Route: `https://consumer-<username>.{{% param labAppUrl %}}/data`

{{% alert  color="primary" %}}Replace **\<username>** with your username or get the URL from your route.{{% /alert %}}

Or you could access the `data` endpoint using curl:

```BASH
curl -kL $({{% param cliToolName %}} get ingress <releasename>-consumer --template="{{(index .spec.rules 0).host}}")/data
```

When you open the URL you should see the producers data

```json
{"data":0.6681209742895893}
```

If you only see `Your new Cloud-Native application is ready!`, then you forgot to append the `/data`path to the URL.


## Task {{% param sectionnumber %}}.4: Prepare another release

At this point we have a configurable Helm chart and a running release. Next we gonna use the cart for another release. We consider to release it into a production environment. therefore we have to adjust some values. First copy the existing `values.yaml` to `values-production.yaml`.

{{< highlight YAML "hl_lines=15" >}}
.
├── Readme.md
└── helm-basic-chart
    ├── Chart.yaml
    ├── templates
    │   ├── helpers.tpl
    │   ├── consumer-deployment.yaml
    │   ├── consumer-ingress.yaml
    │   ├── consumer-service.yaml
    │   ├── producer-deployment.yaml
    │   ├── producer-ingress.yaml
    │   ├── producer-service.yaml
    │   └── tests
    │       └── test-connection.yaml
    ├── values-production.yaml
    └── values.yaml
{{< /highlight >}}


Open the `values-production.yaml` and change following values.

* Debug log level is too high in a production environment, change it to `INFO`
* The resource requirements are usually higher in a production environment than in a development environment. Increase the Memory Limits to `750Mi`
* To avoid DNS collisions we need to chang the host to, change it to `producer-<username>-prod.{{% param labAppUrl %}}` and `consumer-<username>-prod.{{% param labAppUrl %}}`

{{% details title="Solution" %}}


### values-production.yaml


{{< highlight YAML "hl_lines=" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/values-production.yaml" %}}
{{< /highlight >}}


{{% /details %}}


## Task {{% param sectionnumber %}}.6: Install and verify production release


Now we have prepared our values file for the production environment. Next we can install the chart again, but with a different name and different values.
Execute the Helm install command and pass the new created production values as parameter.

```bash
helm upgrade -i myrelease-prod --values values-production.yaml --namespace $USER ./helm-basic-chart
```

Use the helm list command to list all releases in your namespace

```bash
helm ls --namespace $USER
```

You should see following output with de development and the production release

```
NAME            NAMESPACE       REVISION        UPDATED                                         STATUS          CHART                   APP VERSION
myrelease       default         1               2022-05-19 13:26:56.278026261 +0200 CEST        deployed        helm-basic-chart-0.1.0  1.16.0
myrelease-prod  default         1               2022-05-19 13:26:36.570013792 +0200 CEST        deployed        helm-basic-chart-0.1.0  1.16.0
```


## Task {{% param sectionnumber %}}.7: Cleanup

```bash
helm uninstall myrelease --namespace $USER
helm uninstall myrelease-prod --namespace $USER
```
