---
title: "4.1 - Deploy your awesome Application"
weight: 41
---

Using the generated and modified Helm chart, we are going to deploy our own awesome application.

### Task 1: Change the deployment.yml Template and the values.yml File

Our container image has the name `appuio/example-spring-boot:latest` and is a very basic python application. Change the content of your `deployment.yml` and `values.yml` so a pod with the `example-spring-boot` image is started instead of the `nginx` image from the default chart we created with `helm create mychart` in lab 2.

The `appuio/example-spring-boot:latest` application is running on port `8080`. Change the existing `deployment.yaml` accordingly.

After the changes, create or upgrade a release from your template.

{{< collapse mobi "Mobi-specific instructions" danger >}}
Use `docker-registry.mobicorp.ch/puzzle/k8s/kurs/example-spring-boot:latest` as your container image instead of `appuio/example-spring-boot:latest`.
{{< /collapse >}}

{{< collapse solution-1 "Solution Task 1" success >}}

In our `values.yaml` we only have to change the value of `image.repository`:

```yaml
# Default values for test.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: appuio/example-spring-boot
  tag: latest
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

And then our `deployment.yaml` should look like this:

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
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
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

To create a release from our chart, we run the following command within our chart directory:

**Helm 2:**
```bash
$ helm install --name myapp --namespace [USER] ./mychart
```

**Helm 3:**
```bash
$ helm install --namespace [USER] myapp ./mychart
```

This will create a new release with the name `myapp`. If we already had installed a release and wanted to update the existing one, we'd use the following command:

```bash
$ helm upgrade myfirstrelease --namespace [USER] ./mychart
```
{{< /collapse >}}


### Task 2: Create Ingress to access the app

The template folder already has a file for an ingress resource. There are even some variables in `values.yaml` to configure it. Set the correct values for our app and upgrade it.

{{< notice tip >}}
The current values for the ingress depends on the Kubernetes cluster. Ask your instructor for the correct values if you are not sure.
{{< /notice >}}

{{< collapse solution-2 "Solution Task 2" success >}}
The `values.yaml` should look like this:

```yaml
ingress:
  enabled: true
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: YOUR_INGRESS_NAME
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

{{< notice tip >}}
For details on flow control, check out the [Helm Documentation](https://helm.sh/docs/chart_template_guide/control_structures/).
{{< /notice >}}

{{< /collapse >}}
