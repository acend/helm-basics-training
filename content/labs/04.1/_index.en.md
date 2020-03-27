---
title: "4.1 - Deploy your own App"
weight: 41
---

Using the generated Helm Chart, we are going to deploy our own application.


### Task 1: Change deployment.yml Template and values.yml File

Our Container Image has the name `registry.puzzle.ch/dreng/myawesomeapp:latest`. With this, try to change the content of your `deplyment.yml` and `values.yml`.


### Task 2: Add Environment Variables to the deployment.yml Template and values.yml

We need some environment variables in our Deployment. Can you figure out how to add them? The Goal of this Task is to allow a user to set them via values.

Add the following environment variables:

* `DATABASE_HOST`
* `DATABASE_USER`
* `DATABASE_PASSWORD`
* `DATABASE_NAME`

The `DATABASE_PASSWORD` should be read from a Kubernetes Secret



### Solution Task 2:

{{% collapse solution-2 "Solution Task 2" %}}

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "test.fullname" . }}
  labels:
    {{- include "test.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "test.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "test.selectorLabels" . | nindent 8 }}
    spec:
    {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
    {{- end }}
      serviceAccountName: {{ include "test.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
              env:
              - name: DATABASE_HOST
                value: "{{ .Values.database.host }}"
              - name: DATABASE_USER
                value: "{{ .Values.database.user }}"
              - name: DATABASE_NAME
                value: "{{ .Values.database.name }}"
              - name: DATABASE_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: {{ .Values.database.secretName }}
                    key: {{ .Values.database.secretKey }}
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

and your `values.yaml` should contain now:

``` 
database:
  host: myhost
  user: myuser
  name: myname
  secretName: secretName
  secretKey: password
```

and of course, you also need the Kubernetes Secret containing the `DATABASE_PASSWORD`

this could look like:

```
apiVersion: v1
kind: Secret
metadata:
  name: secretName
data:
  password: bXlwYXNzd29yZA==
```

{{% /collapse %}}

