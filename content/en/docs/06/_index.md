---
title: "6. Your own complex Helm chart"
weight: 6
sectionnumber: 6
---

Let's create a bit more complex Chart.

In this Lab we are covering folling topics:

* 7.1 Deploy the Chart and add a database to the app
* 7.2 Template the database resources
* 7.3 Deploy the same app with a database dependency instead of your own database deployment
* 7.4 Writing tests for your Helm Charts
* 7.5 Deploy you app with Helm hooks
* 7.6 Prepare and deploy your app for different environments
* 7.7 Publish your Chart


We already prepared a Helm chart skeleton for this, you can clone the Chart with following command:
`git clone https://github.com/acend/helm-complex-chart.git lab7 && cd lab7`
Let's have a closer look at its directory structure and components. The Chart consists of the following files and folders:

```
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
└── values.yaml
```

Looking at the `templates/` directory, we notice that there already are a few files:

* `NOTES.txt`: The "help text" for your chart which will be displayed to your users when they run `helm install`
* `app-deployment.yaml`: A basic manifest for creating a Kubernetes deployment
* `app-service.yaml`: A basic manifest for creating a service endpoint for your deployment
* `app-ingress.yaml`: A basic manifest for creating a ingress to expose your app deployment
* `_helpers.tpl`: A place to put template helpers that you can re-use throughout the chart
* `tests/`: A directory to put test files for testing the deployed Helm chart

Furthermore the are some resources (prefixed with `mariadb-`) for a database backend which we're going to use in the next lab section.
For now you can ignore them.


{{% alert title="Note" color="info" %}}
For details on chart templating, check out the [Helm's getting started guide](https://helm.sh/docs/chart_template_guide/getting_started/).
{{% /alert %}}


## values.yaml

In the `values.yaml` file we defined our values used in our templates:

```yaml
# Default values for helm-complex-chart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: mychart-<namespace>.{{% param labAppUrl %}}
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: mychart-<namespace>-<appdomain>
      hosts:
        - mychart-<namespace>.{{% param labAppUrl %}}

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}

database:
  enabled: false
```

When instantiating a release from a chart, we can overwrite these values according to our own environment-specific conditions.

So, we can for instance create a `values-dev.yaml` where we keep our development environment values and then use `helm upgrade/install -f values-dev.yaml` to update or instantiate a release for the given environment. A different approach is to keep the files under version control and use branches for the different stages.

{{% alert title="Note" color="info" %}}
For details on the values file, check out the [Helm documentation about values files](https://helm.sh/docs/chart_template_guide/values_files/).
{{% /alert %}}


## Templates

All our Kubernetes resource files are in the `templates` folder. Let's have a closer look at `templates/app-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "helm-complex-chart.fullname" . }}
  labels:
    {{- include "helm-complex-chart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "helm-complex-chart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "helm-complex-chart.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "helm-complex-chart.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

We can see that they look similar to the well-known Kubernetes resource files, but we have some control elements starting and ending with two curly brackets (`{{ }}`). These template files are rendered through a [Go template](https://golang.org/pkg/text/template/) rendering engine. More will be covered on `Go templates` in an upcoming lab.

{{% alert title="Note" color="info" %}}
For details on templating, check out the [Helm documentation about template functions and pipelines](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/).
{{% /alert %}}


### _helpers.tpl

Inside the template folder you can also find a `_helpers.tpl` file.

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "helm-complex-chart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
If release name contains chart name it will be used as a full name.
*/}}
{{- define "helm-complex-chart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "helm-complex-chart.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "helm-complex-chart.labels" -}}
helm.sh/chart: {{ include "helm-complex-chart.chart" . }}
{{ include "helm-complex-chart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "helm-complex-chart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "helm-complex-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "helm-complex-chart.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "helm-complex-chart.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```

As you can see, you can also define [named templates](https://helm.sh/docs/chart_template_guide/named_templates/) in Helm and then use these named templates.

Have a look at:

```yaml
{{- define "helm-complex-chart.labels" -}}
helm.sh/chart: {{ include "helm-complex-chart.chart" . }}
{{ include "helm-complex-chart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
```

you can then access this `helm-complex-chart.labels` in your `app-deployment.yaml` like follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "helm-complex-chart.fullname" . }}
  labels:
    {{- include "helm-complex-chart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
```


### Tests

Helm has the capability to test the deployed Kubernetes resources.
In the `/test` directory we can define multiple test jobs for the deployed Helm chart.
A test job is defined by a Pod resource which specifies a container with a given command to run. If the container exits successfully (exit code 0), the test was successful. Further the test Pod must contain following annotation `helm.sh/hook: test` to run during the Helm deployment.
Inside the `/test` directory you can find already a simple test job. This test job starts a busybox image with a wget command to check if the deployed app is reachable by its service name.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "helm-complex-chart.fullname" . }}-test-connection"
  labels:
    {{- include "helm-complex-chart.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "helm-complex-chart.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

You can execute the tests on a given release with the following command:

```bash
helm test myapp --namespace <namespace>
```

What tests are useful:

* Verify the configuration, eg. username and passwords are correct
* Make sure the deployed application is reachable over the service
* Run functional smoke tests for example logging into an application


## Task {{% param sectionnumber %}}.1: Change Chart.yaml

Study the [Helm documentation about the Chart.yaml file](https://helm.sh/docs/topics/charts/#the-chartyaml-file), then change the description to `My awesome app` and add yourself to the list of maintainers.


### Solution

```yaml
apiVersion: v2
name: helm-complex-chart
description: My awesome app
type: application
version: 0.1.0
appVersion: 1.16.0
maintainers:
  - name: YOUR NAME
    email: YOUR E-MAIL ADDRESS
```

Continue with the lab "[Deploy your awesome application](./deploy/)".
