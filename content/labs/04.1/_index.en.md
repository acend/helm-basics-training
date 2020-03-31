---
title: "4.1 - Deploy your awesome Application"
weight: 41
---

Using the generated Helm Chart, we are going to deploy our own application.


### Task 1: Change deployment.yml Template and values.yml File

Our Container Image has the name `appuio/example-spring-boot:latest` and is a very basic Springboot application. Change the content of your `deployment.yml` and `values.yml` so a Pod with the `example-spring-boot` image is started instead of the `nginx` image from the default chart your created with `helm create mychart` in lab 2.

The `appuio/example-spring-boot:latest` application is running on port `8080`. Change the existing `deployment.yaml` to suit for this.

After the changes, create or upgrade a release from your template.

{{% collapse solution-1 "Solution Task 1" %}}

In your `values.yaml` you currently only have to change `image.repository` with the new image name.

```yaml

# Default values for test.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: appuio/example-spring-boot
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

And then your `deployment.yaml` should look like this:

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
              containerPort: 8080
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

As you see, the Deployment uses `{{ .Chart.AppVersion }}` for the image tage name. So you have to change this to `latest` in your `charts.yaml`

To create a release, run within your chart directory:

```bash
helm install myapp .
```

this will create a new release with the name `myapp`. If you already have installed a release and want to update the existing one, use the following command:

```bash
helm upgrade myreleasename .
```

{{% /collapse %}}


### Task 2: Create Ingress to access the App

The template folder does already have a file for an ingress and there are some variables in `values.yaml` to configure the ingress. Set the correct values for your app and upgrade it.



{{% collapse solution-2 "Solution Task 2" %}}

Your `values.yaml` should look like this:

```yaml
ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: YOUR_INRESS_NAME
      paths:
      - /
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local
```

So `ingress.enabled` is set to `true` and you have to define a `host` for your ingress (this depends on your lab setup, ask your teacher for correct falues). Furthermore, we have to define a `path` to your app. Lets have a look into your ingress template file to understand whats happening with the `host` & `path` array.

```yaml

{{- if .Values.ingress.enabled -}}
{{- $fullName := include "test.fullname" . -}}
{{- $svcPort := .Values.service.port -}}
{{- if semverCompare ">=1.14-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: networking.k8s.io/v1beta1
{{- else -}}
apiVersion: extensions/v1beta1
{{- end }}
kind: Ingress
metadata:
  name: {{ $fullName }}
  labels:
    {{- include "test.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
{{- if .Values.ingress.tls }}
  tls:
  {{- range .Values.ingress.tls }}
    - hosts:
      {{- range .hosts }}
        - {{ . | quote }}
      {{- end }}
      secretName: {{ .secretName }}
  {{- end }}
{{- end }}
  rules:
  {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
        {{- range .paths }}
          - path: {{ . }}
            backend:
              serviceName: {{ $fullName }}
              servicePort: {{ $svcPort }}
        {{- end }}
  {{- end }}
{{- end }}

```

As you see, there is a `{{- range .Values.ingress.hosts }} [...] {{- end }}` which loops trough all the values in the `host` array. The same happens to the `path` value.

{{% /collapse %}}