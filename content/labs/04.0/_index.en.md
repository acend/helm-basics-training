---
title: "4.0 - Create your own Helm Chart"
weight: 40
---

Remember "Lab 2.0 - Create a simple Chart" where you created your first chart? Lets have a closer look at its directory structure and components. A typical chart consists of the following files and folders:

```
./
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

Let's take a closer look at the `mychart/templates/` directory. There's already a few files there:

* `NOTES.txt`: The "help text" for your chart which will be displayed to your users when they run `helm install`
* `deployment.yaml`: A basic manifest for creating a Kubernetes deployment
* `service.yaml`: A basic manifest for creating a service endpoint for your deployment
* `_helpers.tpl`: A place to put template helpers that you can re-use throughout the chart

{{< notice tip >}}
For details on chart templating, check out the [Helm Documentation](https://helm.sh/docs/chart_template_guide/getting_started/).
{{< /notice >}}


### values.yaml

In the `values.yaml` file we define our values used in our templates:

```yaml
# Default values for test.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent

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
  name:

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
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: []
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

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

nodeSelector: {}

tolerations: []

affinity: {}
```

When instantiating a release from a chart, we can overwrite these values which allows us to specify our environment-specific values.

So for instance we can create a `values-dev.yaml` where we keep our development environment values and then use `helm upgrade/install -f values-dev.yaml` to update or instantiate a release for the given environment. A different approach is to keep the files under version control and use branches for the different stages.

{{< notice tip >}}
For details on the values file, check out the [Helm Documentation](https://helm.sh/docs/chart_template_guide/values_files/):
{{< /notice >}}


### Templates

All our Kubernetes resource files are in the `templates` folder. Let's have a closer look at `templates\deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
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
          image: "{{ .Values.image.repository }}:{{ .Chart.AppVersion }}"
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

We can see that they look similar to the well-known Kubernetes resource files, but we have some control elements starting and ending with two curly brackets (`{{ }}`). These template files are rendered through a [Go Template](https://golang.org/pkg/text/template/) rendering engine.

{{< notice tip >}}
For details on templating, check out the [Helm Documentation](https://helm.sh/docs/chart_template_guide/functions_and_pipelines/)
{{< /notice >}}


### Task 1: Change charts.yaml

Check out the [Helm Documentation](https://v2.helm.sh/docs/charts/#the-chart-yaml-file) for the `charts.yaml` file, then change the description to `My Awesome App` and add yourself to the list of maintainers:

{{< collapse solution-1 "Solution Task 1" >}}

```yaml
apiVersion: v1
appVersion: "1.0"
description: My awesome App
name: mychart
version: 0.1.0
maintainers: 
  - name: YOUR NAME
    email: YOUR E-MAIL ADDRESS
```

{{< /collapse >}}
