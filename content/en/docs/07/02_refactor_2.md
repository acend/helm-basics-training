---
title: "7.2 Refactor 2"
weight: 72
sectionnumber: 7.2
---

If you take a closer look at your Chart you will still recognize some weak spots. For example the producer and consumer will look like a lot of code duplication. We don't like code duplication at all! The only big difference is the `"consumer"` or `"producer"` pre- or suffixed everywhere.

**Just for fun:** How much lines of code are actually different?

{{% details title="Hint / Solution" %}}

```bash

bc -l <<< "$(diff -U0 templates/consumer-deployment.yaml templates/producer-deployment.yaml | wc -l)"/2

```

{{% /details %}}

When considering the differences and how they affect the service, we can easily see the flaw of this Chart. The entire Chart is a duplication. Both services could use the same Chart and just be two instances / releases!

There are now two possibilities achieving the reduction of code duplication here: instantiation or composition.


## {{% param sectionnumber %}}.1 Instantiate the Chart two times

The idea is simple: Instead of having a Chart consisting of two deployments, two services and two ingresses, reduce all resources to one! Eliminate the specifics in the variable names (if you like), if you're lazy you can just remove half of the Chart and continue.


### Template files

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


### Values

Update your variables by removing top most yaml-object "consumer" or "producer". So there is only one configuration for one service left in your `values.yaml` file. Add another value called `serviceName` to your `values.yaml`.

Your `values.yaml` should look like this (might differ if you deleted the consumer or producer part):

```yaml
# values.yaml

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
serviceName: 
producerServiceName:

```


### Deployment

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
          {{- if .Values.producerServiceName }}
          - name: DATA_PRODUCER_API_MP_REST_URL
            value: http://{{ .Values.producerServiceName }}:8080
          {{- end}}
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


{{% /details %}}


### Service


{{% details title="Solution" %}}

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


{{% /details %}}


### Ingress

{{% details title="Solution" %}}

**ingress.yaml**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
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
                name: {{ include "helm-basic-chart.fullname" . }}-{{ .Values.serviceName }}
                port:
                  number: 8080
            path: /
            pathType: ImplementationSpecific
  tls:
    - hosts:
        - {{ .Values.host }}
```

{{% /details %}}


## Task {{% param sectionnumber %}}.2 Install release

Install the release with the configuration for the producer. As we learned in previous chapters, we can overwrite values from the `values.yaml` with the help of the `--set variable=value` parameter of the helm-cli. Overwrite the values for the producer like the following:

* `host`: `producer-<username>.{{% param labAppUrl %}}`
* `image.name`: `quay.io/puzzle/quarkus-techlab-data-producer`
* `serviceName`: `producer`

Call the release data-producer and install it!

{{% details title="Solution" %}}

```bash

helm upgrade -i producer helm-basic-chart/. --set host=producer-<username>.{{% param labAppUrl %}} --set image.name=quay.io/puzzle/quarkus-techlab-data-producer --set serviceName=producer

```

{{% /details %}}

After you installed the producer service you can verify the deployment if you'd like to be sure you did everything right!

Let's do the same thing and deploy the consuming service accordingly. Overwrite the following values for the data-consumer microservice:

* `host`: `consumer-<username>.{{% param labAppUrl %}}`
* `image.name`: `puzzle/quarkus-techlab-data-consumer`
* `serviceName`: `data-consumer`
* `producerServiceName` : `producer-helm-basic-chart-producer`

{{% details title="Solution" %}}

```bash
helm upgrade -i consumer helm-basic-chart/. --set host=consumer-<username>.{{% param labAppUrl %}} --set image.name=quay.io/puzzle/quarkus-techlab-data-consumer --set serviceName=data-consumer --set producerServiceName=producer-helm-basic-chart-producer
```

{{% /details %}}

At the end, verify your two releases again and test if they are still delivering data as they did before!

```bash

curl -kL $({{% param cliToolName %}} get ingress <releasename>-consumer --template="{{(index .spec.rules 0).host}}")/data
{"data":0.4145158804475594}

```


## Task {{% param sectionnumber %}}.3 Clean up

Uninstall the two releases again to have a fresh ground for the second option!

```bash

helm uninstall producer
helm uninstall consumer

```
