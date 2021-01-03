---
title: "Your awesome application"
weight: 41
---

Using the generated and modified Helm chart, we are going to deploy our own awesome application.


## Task 1: Adapt the chart

{{% onlyWhenNot mobi %}}
Our container image has the name `quay.io/acend/example-web-python:latest` and is a very basic python application. Change the content of your `deployment.yaml` and `values.yaml` so a pod with the `example-web-python` image is started instead of the `nginx` image from the default chart we created with `helm create mychart` in lab 2.

The `quay.io/acend/example-web-python:latest` application is running on port `5000`. Change the existing `deployment.yaml` accordingly.
{{% /onlyWhenNot %}}

After the changes, create or upgrade a release from your template.

{{% onlyWhen mobi %}}
Our container image has the name `docker-registry.mobicorp.ch/puzzle/k8s/kurs/example-web-python:latest` and is a very basic python application. Change the content of your `deployment.yaml` and `values.yaml` so a pod with the `example-web-python` image is started instead of the `nginx` image from the default chart we created with `helm create mychart` in lab 2.

The `docker-registry.mobicorp.ch/puzzle/k8s/kurs/example-web-python:latest` application is running on port `5000`. Change the existing `deployment.yaml` accordingly.
{{% /onlyWhen %}}


### Solution

In our `values.yaml` we only have to change the value of `image.repository` and `image.tag`:

{{% onlyWhenNot mobi %}}

```yaml
# Default values for mychart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: quay.io/acend/example-web-python
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: latest

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

{{% /onlyWhenNot %}}
{{% onlyWhen mobi %}}

```yaml
# Default values for mychart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: docker-registry.mobicorp.ch/puzzle/k8s/kurs/example-web-python
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: latest

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

{{% /onlyWhen %}}

And then our `deployment.yaml` should look as follows. Note the changed `.spec.containers[0].ports[0].containerPort`:

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
              containerPort: 5000
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

To create a release from our chart, we run the following command within our chart directory:

```bash
helm install myapp ./mychart --namespace <namespace>
```

This will create a new release with the name `myapp`. If we already had installed a release and wanted to update the existing one, we'd use the following command:

```bash
helm upgrade myfirstrelease --namespace <namespace> ./mychart
```


## Task 2: Make the app accessible

The template folder already has a file for an ingress resource. There are even some variables in `values.yaml` to configure it. Set the correct values for our app and upgrade it.

{{% alert title="Warning" color="secondary" %}}
The current values for the ingress depends on the Kubernetes cluster. Ask your instructor for the correct values if you are not sure.
{{% /alert %}}

{{% onlyWhen mobi %}}
You can use `helmtechlab-python-<namespace>.phoenix.mobicorp.test` as your hostname if you want to access your deployed application. It might take some time until your ingress hostname is accessable as the DNS name first has to be propagated correctly.
{{% /onlyWhen %}}


### Solution Task 2

The `values.yaml` should look like this:

```yaml
ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: awesome.<namespace>.<appdomain>
      paths:
      - /
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local
```

So `ingress.enabled` is set to `true` and we have to define a `host` for the ingress resource. Furthermore, we have to define a `path` to our app. Lets have a look into the ingress template file to understand what's happening with the `host` & `path` array.

```yaml
{{- if .Values.ingress.enabled -}}
{{- $fullName := include "mychart.fullname" . -}}
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
    {{- include "mychart.labels" . | nindent 4 }}
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

As we can see there is a `{{- range .Values.ingress.hosts }} [...] {{- end }}` which loops trough all the values in the `host` array. The same happens to the `path` value.

{{% alert title="Note" color="primary" %}}
For details on flow control, check out the [Helm Documentation](https://helm.sh/docs/chart_template_guide/control_structures/).
{{% /alert %}}

Continue with the lab "[A new backend](../database/)".
