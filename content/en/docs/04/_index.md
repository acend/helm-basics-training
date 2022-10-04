---
title: "4. Create your own chart"
weight: 4
sectionnumber: 4
---

In this section were going to show you how to modify a Helm chart from an existing Kubernetes deployment. To provide you an easy entry point, we have already prepared a Helm chart skeleton. This chart contains all necessary template files. With this chart, we want to deploy two java microservices, one which produces random data when it’s REST interface is called. The other microservice consumes then the data and exposes it to its endpoint.

```
+----------+                    +----------+
| producer +<-------------------+ consumer +
+----------+                    +----------+
```

Imagine your being a java development team embracing the DevOps culture and your job is to deploy the two services you designed to your Kubernetes cluster.


## Task {{% param sectionnumber %}}.1: Get the chart skeleton

Clone the repository we have prepared for you containing all the needed template files to get you started.

```bash
git clone https://github.com/acend/helm-basic-chart.git
```

After cloning the chart you have following structure:

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

The template files are always prefixed with the according application (e.g. `consumer-deployment.yaml` and `producer-deployment.yaml`) and suffixed with the resource they represent. This is a best approach we like to follow in order to keep our files ordered by the applications and for readability purposes.

The setup is quite simple here, for each application we have a deployment, a service and an ingress. If you inspect the files in the folder you will realize that we have a lot of hard coded properties we maybe would like to change in order to bring the applications to multiple environments.


## Task {{% param sectionnumber %}}.2: Make the Ingress resource more configurable

After cloning the repository and inspecting the templates already created for you, you will notice some potential for improvement. For example the hostname of the applications should not be hard-coded. When deploying to multiple environments you will run into conflicts.

Modify the **consumer and producer** ingress templates and extract following variables to make them configurable:

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

{{< highlight YAML "hl_lines=11 23" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/templates/consumer-ingress.yaml" %}}
{{< /highlight >}}

Replace the same value in our `producer-ingress.yaml` file.

{{< highlight YAML "hl_lines=11 23" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/templates/producer-ingress.yaml" %}}
{{< /highlight >}}


Afterwards we can install our Helm Chart with following command.

```s
helm install myrelease --namespace <namespace> ./helm-basic-chart
```

Verify your deployment! Check if your pods are running and healthy!

```bash
kubectl get pods
```

This should return something like this:

```
kubectl get pods
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

curl -kL $(kubectl get ingress <releasename>-consumer --template="{{(index .spec.rules 0).host}}")/data
{"data":0.15495350024595755}

```

If your application returns the data point when consuming the consumers `/data` endpoint, then both applications work.

{{% /details %}}


## Task {{% param sectionnumber %}}.3: Make the deployments more configurable

Not just the ingresses could use some improvements. Also the deployments could be more configurable. In order to keep up to date with the current requirements your task is to adapt the following things:

Producer Deployment:

* Extract the image tag from the `.spec.containers[0].image` on Line 22 field as value
* Extract the `.spec.containers[0].resources` block from line 51 as value `consumer.resources`. Make use of the `toYaml` and the `nindent` function.
* Extract the `.spec.containers[0].env["QUARKUS_LOG_LEVEL"]` on line 26 block as value

Consumer Deployment:

* Extract the `.spec.containers[0].resources` block from line 51 as value `producer.resources`. Make use of the `toYaml` and the `nindent` function.
* Extract the `.spec.containers[0].env["QUARKUS_LOG_LEVEL"]` on line 26 block as value `producer.logLevel`

{{% alert title="Note" color="primary" %}}
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


## Task {{% param sectionnumber %}}.4: Upgrade the chart

Execute following command to update our helm release.

```shell
helm upgrade myrelease --namespace <namespace> ./helm-basic-chart
```

Finally, you can visit your application with the URL provided from the Route: `https://consumer-<username>.labapp.acend.ch/data`

{{% alert  color="primary" %}}Replace **\<username>** with your username or get the URL from your route.{{% /alert %}}

Or you could access the `data` endpoint using curl:

```BASH
curl -kL $(kubectl get ingress <releasename>-consumer --template="{{(index .spec.rules 0).host}}")/data
```

When you open the URL you should see the producers data

```json
{"data":0.6681209742895893}
```

If you only see `Your new Cloud-Native application is ready!`, then you forgot to append the `/data`path to the URL.


## Task {{% param sectionnumber %}}.5: Prepare another release

At this point we have a configurable Helm chart and a running release. Next we gonna use the cart for another release. We consider to release it into a productive environment. therefore we have to adjust some values. First copy the existing `values.yaml` to `values-productive.yaml`.
Open the `values-productive.yaml` and change following values.

* Debug log level is too high in a productive environment, change it to `INFO`
* The resource requirements are usually higher in a productive environment than in a development environment. Increase the Memory Limits to `750Mi`
* To avoid DNS collisions we need to chang the host to, change it to `producer-<username>-prod.labapp.acend.ch` and `consumer-<username>-prod.labapp.acend.ch`


## Solution Task {{% param sectionnumber %}}.5


### values-production.yaml

{{< highlight YAML "hl_lines=" >}}
{{% remoteFile "https://raw.githubusercontent.com/acend/helm-basic-chart/solution/helm-basic-chart/values-production.yaml" %}}
{{< /highlight >}}


## Task {{% param sectionnumber %}}.6: Install and verify production release

Now we have prepared our values file for the production environment. Next we can install the chart again, but with a different name and different values.
Execute the Helm install command and pass the new created production values as parameter.

```bash
helm install myrelease-prod --values values-production.yaml --namespace <namespace> ./helm-basic-chart
```

Use the helm list command to list all releases in your namespace

```bash
helm ls --namespace <namespace>
```

You should see following output with de development and the production release

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


## Task {{% param sectionnumber %}}.8: Refactoring

If you take a closer look at your Chart you will still recognize some weak spots. For example the producer and consumer will look like a lot of code duplication. We don't like code duplication at all! The only big difference is the `"consumer"` or `"producer"` pre- or suffixed everywhere.

**Just for fun:** How much lines of code are actually different?

{{% details title="Hint / Solution" %}}

```bash

bc -l <<< "$(diff -U0 templates/consumer-deployment.yaml templates/producer-deployment.yaml | wc -l)"/2

```

{{% /details %}}

When considering the differences and how they affect the service, we can easily see the flaw of this Chart. The entire Chart is a duplication. Both services could use the same Chart and just be two instances / releases!

There are now two possibilities achieving the reduction of code duplication here: instantiation or composition.


## {{% param sectionnumber %}}.8.1: Option 1: Instantiate the Chart two times

The idea is simple: Instead of having a Chart consisting of two deployments, two services and two ingresses, reduce all resources to one! Eliminate the specifics in the variable names (if you like), if you're lazy you can just remove half of the Chart and continue.


### Task {{% param sectionnumber %}}.8.1.1

Start of by removing all the resources for the one of the two services and rename them by removing the prefix "producer" or "consumer". After doing so, your Chart's structure should look something like this:

```bash

$ tree

.
├── helm-basic-chart
│   ├── Chart.yaml
│   ├── templates
│   │   ├── deployment.yaml
│   │   ├── _helpers.tpl
│   │   ├── ingress.yaml
│   │   ├── service.yaml
│   │   └── tests
│   │       └── test-connection.yaml
│   └── values.yaml
└── Readme.md

```


### Task {{% param sectionnumber %}}.8.1.2

Update your variables by removing top most yaml-object "consumer" or "producer". So there is only one configuration for one service left in your `values.yaml` file. Add another value called `serviceName` to your `values.yaml`.

Your `values.yaml` should look like this (might differ if you deleted the consumer or producer part):

```yaml
# values.yaml

host: consumer-user4.labapp.acend.ch
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
serviceName: 

```


### Task {{% param sectionnumber %}}.8.1.3

Update your deployment, service and ingress and edit the values accordingly. So your `{{ .Values.producer.image.tag }}` will become `{{ .Values.image.tag }}`.

Change the hard-coded occurrences of `data-producer` or `data-consumer` in your templates to `{{ .Values.serviceName }}`. If you fancy you can remove the suffixes `-producer` or `-consumer` in your templates as well.

{{% details title="Solution" %}}

**deployment.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
  name: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
      app: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        deployment: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
        app: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
    spec:
      containers:
        - image: {{ .Values.image.name }}:{{ .Values.image.tag }}
          imagePullPolicy: Always
          env:
          - name: QUARKUS_LOG_LEVEL
            value: {{ .Values.logLevel }}
          - name: DATA_PRODUCER_API_MP_REST_URL
            value: http://{{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}:8080
          livenessProbe:
            failureThreshold: 5
            httpGet:
              path: /health/live
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 3
            periodSeconds: 20
            timeoutSeconds: 15
          readinessProbe:
            failureThreshold: 5
            httpGet:
              path: /health/ready
              port: 8080
              scheme: HTTP
            initialDelaySeconds: 3
            periodSeconds: 20
            timeoutSeconds: 15
          name: {{ .Values.serviceName }}
          ports:
            - containerPort: 8080
              name: http
              protocol: TCP
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

**service.yaml**:
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
  name: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
spec:
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    deployment: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
  sessionAffinity: None
  type: ClusterIP
```

**ingress.yaml**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/tls-acme: "true"
  labels:
    app: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
  name: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
spec:
  rules:
    - host: {{ .Values.host }}
      http:
        paths:
          - backend:
              service:
                name: {{ .Values.serviceName }}
                port:
                  number: 8080
            path: /
            pathType: ImplementationSpecific
  tls:
    - hosts:
        - {{ .Values.host }}
      secretName: producer-labapp-acend-ch
```

{{% /details %}}


### Task {{% param sectionnumber %}}.8.1.4

Install the release with the configuration for the producer. As we learned in previous chapters, we can overwrite values from the `values.yaml` with the help of the `--set variable=value` parameter of the helm-cli. Overwrite the values for the producer like the following:

* `host`: `producer-user4.labapp.acend.ch`
* `image.name`: `quay.io/puzzle/quarkus-techlab-data-producer`
* `serviceName`: `producer`

Call the release data-producer and install it!

{{% details title="Solution" %}}

```bash

helm install producer helm-basic-chart/. --set host=producer-user4.labapp.acend.ch --set image.name=quay.io/puzzle/quarkus-techlab-data-producer --set serviceName=producer

```

{{% /details %}}

After you installed the producer service you can verify the deployment if you'd like to be sure you did everything right!

Let's do the same thing and deploy the consuming service accordingly. Overwrite the following values for the data-consumer microservice:

* `host`: `consumer-user4.labapp.acend.ch`
* `image.name`: `puzzle/quarkus-techlab-data-consumer`
* `serviceName`: `data-consumer`

{{% details title="Solution" %}}

```bash

helm install consumer helm-basic-chart/. --set host=consumer-user4.labapp.acend.ch --set image.name=quay.io/puzzle/quarkus-techlab-data-consumer --set serviceName=data-consumer

```

{{% /details %}}

At the end, verify your two releases again and test if they are still delivering data as they did before!

```bash

curl -kL $(kubectl get ingress <releasename>-consumer --template="{{(index .spec.rules 0).host}}")/data
{"data":0.4145158804475594}

```


### Task {{% param sectionnumber %}}.8.1.5

Uninstall the two releases again to have a fresh ground for the second option!

```bash

helm uninstall producer
helm uninstall consumer

```


## {{% param sectionnumber %}}.8.2: Option 2: Composition

Instead of instantiating the Chart two times manually by hand, we can also define a composition of two Charts in a single Chart. In other words we create a new Helm Chart and add the Chart we just wrote twice as a dependency to the chart.

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


### Task {{% param sectionnumber %}}.8.2.1

Create a new Chart `parent` and empty the templates folder, since we don't need any additional templates in the Chart other than the dependencies.

{{% details title="Hint" %}}

```bash

helm create parent
rm -rf parent/templates/*

```

{{% /details %}}


### Task {{% param sectionnumber %}}.8.2.2

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


### Task {{% param sectionnumber %}}.8.2.3

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


### Task {{% param sectionnumber %}}.8.2.4

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
  host: consumer-<username>.labapp.acend.ch
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

producer:
  host: producer-<username>.labapp.acend.ch
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


### Task {{% param sectionnumber %}}.8.2.5

Install a Helm Chart release `myrelease` and verify if the two services are running correctly!

{{% details title="Solution" %}}

```bash

helm install myrelease parent/.

```

```bash

curl -kL $(kubectl get ingress <releasename>-consumer --template="{{(index .spec.rules 0).host}}")/data
{"data":0.4145158804475594}

```

{{% /details %}}


### Task {{% param sectionnumber %}}.8.2.6

Congratulations! You succeeded in the Chapter and the only thing left is to do some cleanup afterwards!

Remove all the installed objects installed during the chapter!

```bash

helm uninstall myrelease

```
