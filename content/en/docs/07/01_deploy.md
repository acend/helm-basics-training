---
title: "7.1 Deploy your awesome application"
weight: 71
sectionnumber: 7.1
---

Using the generated and modified Helm chart, we are going to deploy our own awesome application.


## Task {{% param sectionnumber %}}.1: Deploy the chart

Let's us deploy our awesome application. Therefore we need to adjust the values file.

```yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: mychart-<namespace>.labapp.acend.ch
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: mychart-<namespace>-<appdomain>
      hosts:
        - mychart-<namespace>.labapp.acend.ch
```


### Solution



{{% onlyWhenNot mobi %}}
{{% alert title="Note" color="primary" %}}
Don't forget to replace `{{% param labAppUrl %}}` with the value provided by the trainer.
{{% /onlyWhenNot %}}
{{% onlyWhen mobi %}}
It might take some time until your ingress hostname is accessible, as the DNS name first has to be propagated correctly.
{{% /onlyWhen %}}
{{% /alert %}}


To create a release from our chart, we run the following command within our chart directory:

```bash
helm upgrade -i myapp ./mychart --namespace <namespace>
```

This will create a new release with the name `myapp`. If we already had installed a release and wanted to update the existing one, we'd use the following command:

```bash
helm upgrade -i myapp  ./mychart --namespace <namespace>
```

Check whether the ingress was successfully deployed by accessing the URL `http://mychart-<namespace>.{{% param labAppUrl %}}/`


## Task {{% param sectionnumber %}}.2: Connect to the database

In order for the python application to be able to connect to the newly deployed database, we need to add some environment variables to our deployment. The goal of this task is to allow a user to set them via values.

Add the following environment variables:

* `MYSQL_DATABASE_NAME` reference the database username value from the secret you created in task 1
* `MYSQL_DATABASE_PASSWORD` reference the database password value from the secret you created in task 1
* `MYSQL_DATABASE_ROOT_PASSWORD` reference the the database root password value from the secret you created in task 1
* `MYSQL_DATABASE_USER` reference the database user value from the secret you created in task 1
* `MYSQL_URI` with the value `mysql://$(MYSQL_DATABASE_USER):$(MYSQL_DATABASE_PASSWORD)@{{ .Release.Name }}-mariadb/$(MYSQL_DATABASE_NAME)`


### Solution Task {{% param sectionnumber %}}.2

Change your Application Deployment Template in `templates/deployment.yaml` and include the new environment variables:

{{< highlight YAML "hl_lines=36-52" >}}
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
          env:
          - name: MYSQL_DATABASE_USER
            value: {{ .Values.database.databaseuser }}
          - name: MYSQL_DATABASE_PASSWORD
            valueFrom:
              secretKeyRef:
                key: mariadb-password
                name: {{ .Release.Name }}-mariadb
          - name: MYSQL_DATABASE_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                key: mariadb-root-password
                name: {{ .Release.Name }}-mariadb
          - name: MYSQL_DATABASE_NAME
            value: {{ .Values.database.databasename }}
          - name: MYSQL_URI
            value: mysql://$(MYSQL_DATABASE_USER):$(MYSQL_DATABASE_PASSWORD)@{{ .Release.Name }}-mariadb/$(MYSQL_DATABASE_NAME)
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
{{< /highlight >}}

Add following changes in the `values.yaml` to enable and configure the database:

```yaml
database:
  enabled: true
  databaseuser: acend
  databasename: acenddb
  databasepassword: mysuperpassword123
  databaserootpassword: mysuperrootpassword123
  image:
    repository: mariadb
    pullPolicy: IfNotPresent
    tag: "10.5"
```

To upgrade your existing release run:

```bash
helm upgrade myapp ./mychart --namespace <namespace>
```


## Task {{% param sectionnumber %}}.8: Clean up

Uninstall the app

```bash
helm uninstall myapp --namespace <namespace>
```


