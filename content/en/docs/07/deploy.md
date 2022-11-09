---
title: "7.1 Your awesome application"
weight: 71
sectionnumber: 7.1
---

Using the generated and modified Helm chart, we are going to deploy our own awesome application.


## Task {{% param sectionnumber %}}.1: Adapt the chart

{{% onlyWhenNot mobi %}}
Our container image has the name `quay.io/acend/example-web-python:latest` and is a very basic python application. Change the content of your `deployment.yaml` and `values.yaml` so that a pod with the `example-web-python` image is started instead of the `nginx` image from the default chart we created with `helm create mychart` in lab 1.

The `quay.io/acend/example-web-python:latest` application is running on port `5000`. Change the existing `deployment.yaml` accordingly.
{{% /onlyWhenNot %}}

After the changes, create or upgrade a release from your template.

{{% onlyWhen mobi %}}
Our container image has the name `<registry-url>/puzzle/k8s/kurs/example-web-python:latest` and is a very basic python application. Change the content of your `deployment.yaml` and `values.yaml` so that a pod with the `example-web-python` image is started instead of the `nginx` image from the default chart we created with `helm create mychart` in lab 1.

The `<registry-url>/puzzle/k8s/kurs/example-web-python:latest` application is running on port `5000`. Change the existing `deployment.yaml` accordingly.
{{% /onlyWhen %}}


### Solution

In our `values.yaml` we only have to change the value of `image.repository` and `image.tag`:


{{% onlyWhenNot mobi %}}
{{% onlyWhen openshift %}}

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
  enabled: true
  annotations:
    # kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: mychart-<namespace>.{{% param labAppUrl %}}
      paths:
        - path: /
          pathType: Prefix
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
```

{{% /onlyWhen %}}
{{% onlyWhenNot openshift %}}

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
```

{{% /onlyWhenNot %}}
{{% /onlyWhenNot %}}
{{% onlyWhen mobi %}}

```yaml
# Default values for mychart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: <registry-url>/puzzle/k8s/kurs/example-web-python
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
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: mychart-<namespace>.{{% param labAppUrl %}}
      paths:
      - path: /
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

{{% onlyWhenNot mobi %}}
{{% alert title="Note" color="primary" %}}
Don't forget to replace `{{% param labAppUrl %}}` with the value provided by the trainer.
{{% /onlyWhenNot %}}
{{% onlyWhen mobi %}}
It might take some time until your ingress hostname is accessible, as the DNS name first has to be propagated correctly.
{{% /onlyWhen %}}
{{% /alert %}}

Afterwards our `deployment.yaml` should look as follows. Note the changed `.spec.containers[0].ports[0].containerPort`:

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
helm upgrade -i myapp ./mychart --namespace <namespace>
```

This will create a new release with the name `myapp`. If we already had installed a release and wanted to update the existing one, we'd use the following command:

```bash
helm upgrade <existingrelease> --namespace <namespace> ./mychart
```

Check whether the ingress was successfully deployed by accessing the URL `http://mychart-<namespace>.{{% param labAppUrl %}}/`

Continue with the lab "[A new backend](../database/)".
