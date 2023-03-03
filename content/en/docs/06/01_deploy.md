---
title: "6.1 Deploy your awesome application"
weight: 61
sectionnumber: 6.1
---

Using the generated and modified Helm chart, we are going to deploy our own awesome application.


## Task {{% param sectionnumber %}}.1: Deploy the chart

Let's deploy our awesome application. Therefore we need to adjust the values file.

```yaml
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: helm-complex-chart-<namespace>.labapp.acend.ch
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: helm-complex-chart-<namespace>-<appdomain>
      hosts:
        - helm-complex-chart-<namespace>.labapp.acend.ch
```


### Solution


{{% onlyWhenNot mobi %}}
{{% alert title="Note" color="info" %}}
Don't forget to replace `{{% param labAppUrl %}}` with the value provided by the trainer.
{{% /onlyWhenNot %}}
{{% onlyWhen mobi %}}
It might take some time until your ingress hostname is accessible, as the DNS name first has to be propagated correctly.
{{% /onlyWhen %}}
{{% /alert %}}


To create a release from our chart, we run the following command within our chart directory:

```bash
helm install myapp ./helm-complex-chart --namespace <namespace>
```

This will create a new release with the name `myapp`. If we already had installed a release and wanted to update the existing one, we'd use the following command:

```bash
helm upgrade -i myapp  ./helm-complex-chart --namespace <namespace>
```

Check whether the ingress was successfully deployed by accessing the URL `http://helm-complex-chart-<namespace>.{{% param labAppUrl %}}/`


## Task {{% param sectionnumber %}}.2: Connect to the database

In order for the python application to be able to connect to the newly deployed database, we need to add some environment variables to our deployment. The goal of this task is to allow a user to set them via values.
Change your Application Deployment Template in `templates/app-deployment.yaml` and include the new environment variables:

* `MYSQL_DATABASE_NAME` with the value `acenddb`
* `MYSQL_DATABASE_PASSWORD` references the database password value from the secret you created in task 1
* `MYSQL_DATABASE_ROOT_PASSWORD` references the the database root password value from the secret you created in task 1
* `MYSQL_DATABASE_USER` with the value `acend`
* `MYSQL_URI` with the value `mysql://$(MYSQL_DATABASE_USER):$(MYSQL_DATABASE_PASSWORD)@{{ .Release.Name }}-mariadb/$(MYSQL_DATABASE_NAME)`


### Solution Task {{% param sectionnumber %}}.2


{{< highlight YAML "hl_lines=36-54" >}}
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
          {{- if .Values.database.enabled }}
          env:
          - name: MYSQL_DATABASE_USER
            value: "acend"
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
            value: "acenddb"
          - name: MYSQL_URI
            value: mysql://$(MYSQL_DATABASE_USER):$(MYSQL_DATABASE_PASSWORD)@{{ .Release.Name }}-mariadb/$(MYSQL_DATABASE_NAME)
          {{- end }}
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
{{< /highlight >}}

Add the following changes in the `values.yaml` to enable and configure the database:

```yaml
database:
  enabled: true
```

Enabling this property will render the environment variable block in the deployment template, as well all the `mariadb-*.yaml` templates.
To upgrade your existing release run:

```bash
helm upgrade myapp ./helm-complex-chart --namespace <namespace>
```


## Task {{% param sectionnumber %}}.3: Check


Check whether the attachment of the new backend worked by either looking at the Pod's logs: In there the application tells you which backend it uses, this should of course be the database. Or simply access the application in your browser, create an entry, re-deploy the application Pod (e.g. by scaling it down and up again) and check if your entry is still there.


### Solution Task {{% param sectionnumber %}}.3


First we need to execute the following command to determine the pod name
```bash
{{% param cliToolName %}} get pods -n <namespace>
```

{{< highlight bash "hl_lines=3" >}}
NAME                                    READY   STATUS      RESTARTS   AGE
myapp-helm-complex-chart-test-connection          0/1     Completed   0          22m
myapp-helm-complex-chart-7cc85f99db-n4lsc          1/1     Running     0          12m
myapp-helm-complex-chart-mariadb-74ddcc878-268ts   1/1     Running     0          12m
webshell-67f4cf8c59-st4rg               2/2     Running     0          2d22h
{{< /highlight >}}

Next execute the following command to show the logs
```bash
{{% param cliToolName %}} logs myapp-helm-complex-chart-7cc85f99db-n4lsc
```


As you can see in the log output, our application is now connected to the freshly deployed database
> Never log sensitive informations like database connection strings which contain the password in plain text!

```bash
Using DB:  mysql://acend:mysuperpassword123@myapp-helm-complex-chart-mariadb/acenddb
 * Serving Flask app 'run' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
2021-10-08 11:17:17,783 WARNING:  * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
2021-10-08 11:17:17,784 INFO :  * Running on http://10.42.5.2:5000/ (Press CTRL+C to quit)
2021-10-08 11:17:20,688 INFO : 185.79.235.174 - - [08/Oct/2021 11:17:20] "GET / HTTP/1.1" 200 -
2021-10-08 11:17:21,758 INFO : 185.79.235.174 - - [08/Oct/2021 11:17:21] "GET / HTTP/1.1" 200 -
```


## Task {{% param sectionnumber %}}.8: Clean up


Uninstall the app

```bash
helm uninstall myapp --namespace <namespace>
```
