---
title: "4. Your own Helm chart"
weight: 4
sectionnumber: 4
---

Remember [lab 2 "A simple chart"](../02/) where you created your first chart? Let's have a closer look at its directory structure and components. A typical chart consists of the following files and folders:

```
./
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

Looking at the `mychart/templates/` directory, we notice that there already are a few files:

* `NOTES.txt`: The "help text" for your chart which will be displayed to your users when they run `helm install`
* `deployment.yaml`: A basic manifest for creating a Kubernetes deployment
* `service.yaml`: A basic manifest for creating a service endpoint for your deployment
* `_helpers.tpl`: A place to put template helpers that you can re-use throughout the chart

{{% alert title="Note" color="primary" %}}
For details on chart templating, check out the [Helm's getting started guide](https://helm.sh/docs/chart_template_guide/getting_started/).
{{% /alert %}}


## values.yaml

In the `values.yaml` file we define our values used in our templates:

```yaml
# Default values for mychart.
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
    - host: mychart-<namespace>.<appdomain>
      paths:
        - path: /
  tls:
    - secretName: mychart-<namespace>-<appdomain>
      hosts:
        - mychart-<namespace>.<appdomain>

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
```

When instantiating a release from a chart, we can overwrite these values according to our own environment-specific conditions.

So, we can for instance create a `values-dev.yaml` where we keep our development environment values and then use `helm upgrade/install -f values-dev.yaml` to update or instantiate a release for the given environment. A different approach is to keep the files under version control and use branches for the different stages.

{{% alert title="Note" color="primary" %}}
For details on the values file, check out the [Helm documentation about values files](https://helm.sh/docs/chart_template_guide/values_files/).
{{% /alert %}}


## Templates

All our Kubernetes resource files are in the `templates` folder. Let's have a closer look at `templates/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "mychart.serviceAccountName" . }}
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

{{% alert title="Note" color="primary" %}}
For details on templating, check out the [Helm documentation about template functions and pipelines](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/).
{{% /alert %}}


### _helpers.tpl

Inside the template folder you can also find a `_helpers.tpl` file.

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "mychart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
If release name contains chart name it will be used as a full name.
*/}}
{{- define "mychart.fullname" -}}
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
{{- define "mychart.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "mychart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "mychart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Create the name of the service account to use
*/}}
{{- define "mychart.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "mychart.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```

As you can see, you can also define [named templates](https://helm.sh/docs/chart_template_guide/named_templates/) in Helm and then use these named templates.

Have a look at:

```yaml
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
```

you can then access this `mychart.labels` in your `deployment.yaml` like follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
```


## Task {{% param sectionnumber %}}.1: Change Chart.yaml

Study the [Helm documentation about the Chart.yaml file](https://helm.sh/docs/topics/charts/#the-chartyaml-file), then change the description to `My awesome app` and add yourself to the list of maintainers.


### Solution

```yaml
apiVersion: v2
name: mychart
description: My awesome app
type: application
version: 0.1.0
appVersion: 1.16.0
maintainers:
  - name: YOUR NAME
    email: YOUR E-MAIL ADDRESS
```

Continue with the lab "[Your awesome application](./deploy/)".
