---
title: "4. Create your own chart"
weight: 4
sectionnumber: 4
---

In this section were going to show you how to modify a Helm chart from an exisiting Kubernetes deployment. To provide you an easy entry point, we have already prepared a Helm chart skeleton. This chart contains all necessary template files. With this chart, we want to deploy tow microservices, one which produces random data when it’s REST interface is called. The other microservice consumes then the data and exposes it to its endpoint.

```
+----------+                    +----------+
| producer +<-------------------+ consumer +
+----------+                    +----------+
```


## Task {{% param sectionnumber %}}.1: Get the chart skeleton

```bash
git clone https://github.com/acend/helm-basic-chart.git
```

After cloning the chart you have following structure. The Helm template files are already there.

```
.
├── Readme.md
└── helm-basic-chart
    ├── Chart.yaml
    ├── templates
    │   ├── _helpers.tpl
    │   ├── consumer-deployment.yaml
    │   ├── consumer-ingress.yaml
    │   ├── consumer-service.yaml
    │   ├── producer-deployment.yaml
    │   ├── producer-ingress.yaml
    │   ├── producer-service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml
```


## Task {{% param sectionnumber %}}.2: Make the Ingress resource more configurable

After cloning the Helm basic chart, we can start to modify the chart. Add following modifications:

Modify the **consumer and producer** ingress templates and extract following variables to make them configurable

* Extract `.spec.rules.host` as value
* Extract `.spec.tls.hosts[0]` as value, use the same value as above


{{% alert title="Note" color="primary" %}}
If you want to test the chart locally you can execute following command

`helm template -s templates/consumer-ingress.yaml ./helm-basic-chart --debug | cat -n -`
{{% /alert %}}

First let us define the new variables in our `values.yaml` file. Replace `<username>` with your username

{{< highlight YAML "hl_lines=2 6" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/main/helm-basic-chart/values.yaml" %}}
{{< /highlight >}}

Next replace the hard coded values for the host value in our `consumer-ingress.yaml` file.

{{< highlight YAML "hl_lines=9 21" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/templates/consumer-ingress.yaml" %}}
{{< /highlight >}}

Replace the same value in our `producer-ingress.yaml` file.

{{< highlight YAML "hl_lines=9 21" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/templates/producer-ingress.yaml" %}}
{{< /highlight >}}


Afterwards we can install our Helm Chart with following command.

```
helm install myrelease --namespace <namespace> ./helm-basic-chart
```

## Task {{% param sectionnumber %}}.3: Make the deployments more configurable


Make following values configurable
Producer Deployment:

* Extract the image tag from the `.spec.containers[0].image` on Line 22 field as value
* Extract the `.spec.containers[0].resources`   block from line 51 as value `consumer.resources`. Make use of the `toYaml` and the `nindent` function.
* Extract the `.spec.containers[0].env["QUARKUS_LOG_LEVEL"]` on line 26 block as value
  

Consumer Deployment:

* Extract the `.spec.containers[0].resources`   block from line 51 as value `producer.resources`. Make use of the `toYaml` and the `nindent` function.
* Extract the `.spec.containers[0].env["QUARKUS_LOG_LEVEL"]` on line 26 block as value `producer.logLevel`


## Solution Task {{% param sectionnumber %}}.3


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


## Task {{% param sectionnumber %}}.4: Upgrade the chart

Execute following command to udate our helm release.

`helm upgrade myrelease --namespace <namespace>`

Finally, you can visit your application with the URL provided from the Route: `https://consumer-<username>.labapp.acend.ch/data`

{{% alert  color="primary" %}}Replace **\<username>** with your username or get the URL from your route.{{% /alert %}}

Or you could access the `data` endpoint using curl:

```BASH
curl https://consumer-<username>.labapp.acend.ch/data
```

When you open the URL you should see the producers data

```json
{"data":0.6681209742895893}
```

If you only see `Your new Cloud-Native application is ready!`, then you forget to append the `/data`path to the URL.


## Task {{% param sectionnumber %}}.5: Prepare another release

At this point we have a configurable Helm chart and a running release. Next we gonna use the cart for another release. We consider to release it into a productive environment. therefore we have to adjust some values. First copy the existing `values.yaml` to `values-productive.yaml`.
Open the `values-productive.yaml` and change following values.

* For a productive environment it is a bad practise to use the `latest` tag on a image. Pin the image to the `v1.0.0` tag.
* Debug log level is too high in a productive environment, change it to `INFO`
* The resource requirements are usually higher in a productive environment than in a development environment. Increase the Memory Limits to `750Mi`
* To avoid DNS collisions we need to chang the host to, change it to `producer-<username>-prod.labapp.acend.ch` and `consumer-<username>-prod.labapp.acend.ch`


## Solution Task {{% param sectionnumber %}}.5


### values-production.yaml

{{< highlight YAML "hl_lines=" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/values-production.yaml" %}}
{{< /highlight >}}


## Task {{% param sectionnumber %}}.6: Install and verify production release

Now we have preapred our values file for the production environment. Next we can install the chart again, but with a different name and different values.
Execute the Helm install command and pass the new created production values as paramter.

```bash
helm install myrelease-prod --values values-production.yaml --namespace <namespace> ./helm-basic-chart
```

Use the helm list command to list all releases in your namespace

```bash
helm ls --namespace <namespace>
```

You should see following output with de development and the productoin release
```
NAME            NAMESPACE       REVISION        UPDATED                                         STATUS          CHART                   APP VERSION
myrelease       default         1               2022-05-19 13:26:56.278026261 +0200 CEST        deployed        helm-basic-chart-0.1.0  1.16.0     
myrelease-prod  default         1               2022-05-19 13:26:36.570013792 +0200 CEST        deployed        helm-basic-chart-0.1.0  1.16.0   
```


## Task {{% param sectionnumber %}}.7: Cleanup

```bash
helm delete myrelease --namespace <namespace>
helm delete myrelease-prod --namespace <namespace>
```
