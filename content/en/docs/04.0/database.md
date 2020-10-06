---
title: "A new backend"
weight: 42
---

## Task 1: Create template files

We want to add a MySQL database and use it as a backend for our `example-spring-boot` application. Using the following Kubernetes resource file, create a new template file for the MySQL database:

{{< onlyWhenNot mobi  >}}

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-mysql
  labels:
    app: example-spring-boot
spec:
  selector:
    matchLabels:
      app: example-spring-boot
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: example-spring-boot
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_DATABASE
          value: springboot
        - name: MYSQL_USER
          value: springboot
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-password
              key: password
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-password
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        emptyDir: {}
```

{{< /onlyWhenNot >}}
{{< onlyWhen mobi  >}}
You have to use `docker-registry.mobicorp.ch/puzzle/k8s/kurs/mysql:5.6` as the container image.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: springboot-mysql
  labels:
    app: example-spring-boot
spec:
  selector:
    matchLabels:
      app: example-spring-boot
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: example-spring-boot
        tier: mysql
    spec:
      containers:
      - image: docker-registry.mobicorp.ch/puzzle/k8s/kurs/mysql:5.6
        name: mysql
        env:
        - name: MYSQL_DATABASE
          value: springboot
        - name: MYSQL_USER
          value: springboot
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-password
              key: password
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-password
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        emptyDir: {}
```

{{< /onlyWhen >}}

You also have to create a template for the `mysql-root-password` secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-password
data:
  password: bXlwYXNzd29yZA==
  root-password: bXlwYXNzd29yZA==
```

When creating the template files, make sure that a user can specify the `password` and `root-password` from the secret using a variable.

To connect to the database, we also need a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: springboot-mysql
  labels:
    app: example-spring-boot
spec:
  ports:
    - port: 3306
      targetPort: mysql
  selector:
    app: example-spring-boot
    tier: mysql
  clusterIP: None
```

Have a look at the already existing file `service.yaml` and use this as a starting point for your `springboot-mysql` service.

When your changes are ready, upgrade the already deployed release with the new version.


### Solution

The template file for the MySQL database `templates/deploy-mysql.yaml` could look like this:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}-mysql
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}-mysql
    app.kubernetes.io/instance: {{ .Release.Name }}
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "mychart.name" . }}-mysql
      app.kubernetes.io/instance: {{ .Release.Name }}
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "mychart.name" . }}-mysql
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_DATABASE
          value: "{{ .Values.database.dbname }}"
        - name: MYSQL_USER
          value: "{{ .Values.database.user }}"
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: "{{ .Values.database.secretName }}"
              key: "{{ .Values.database.secretKey }}"
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: "{{ .Values.database.secretName }}"
              key: "{{ .Values.database.secretKeyroot }}"
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        emptyDir: {}

```

The secret `templates/mysql-secret.yaml` file should look like this:

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-password
data:
  password: {{ .Values.database.password | b64enc }}
  rootpassword: {{ .Values.database.rootpassword | b64enc}}
```

Notice the `| b64enc`, which is a builtin function to encode strings with base64.

The service for our MySQL database should look like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: springboot-mysql
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  type: ClusterIP
  ports:
    - port: 3306
      protocol: TCP
  selector:
    app.kubernetes.io/name: {{ include "mychart.name" . }}-mysql
    app.kubernetes.io/instance: {{ .Release.Name }}
```

And then in our `values.yaml` we need to add:

```yaml
database:
  host: myhost
  user: springboot
  dbname: springboot
  secretName: mysql-root-password
  secretKey: password
  secretKeyroot: rootpassword
  password: mysuperpassword123
  rootpassword: mysuperrootpassword123
```

Finally, to upgrade the existing release run:

```bash
helm upgrade myapp --namespace [USER] ./mychart
```


## Task 2: Add environment variables

We need some environment variables in our deployment. The goal of this task is to allow a user to set them via values.

Add the following environment variables:

* `SPRING_DATASOURCE_USERNAME` with value `springboot`
* `SPRING_DATASOURCE_PASSWORD` value from secret you created in task 1
* `SPRING_DATASOURCE_DRIVER_CLASS_NAME` with value `com.mysql.jdbc.Driver`
* `SPRING_DATASOURCE_URL` with value `jdbc:mysql://springboot-mysql/springboot?autoReconnect=true`


### Solution Taks 2

Change your `template/deployment.yml` and include the new environment variables:

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
      app.kubernetes.io/name: {{ include "mychart.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "mychart.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
    {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
    {{- end }}
      serviceAccountName: {{ template "mychart.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
            - name: SPRING_DATASOURCE_URL
              value: "{{ .Values.database.url }}"
            - name: SPRING_DATASOURCE_USERNAME
              value: "{{ .Values.database.user }}"
            - name: SPRING_DATASOURCE_DRIVER_CLASS_NAME
              value: "{{ .Values.database.driver }}"
            - name: SPRING_DATASOURCE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ .Values.database.secretName }}
                  key: {{ .Values.database.secretKey }}
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

Your `values.yaml` should now contain (including the previously defined values from task 2):

```yaml
database:
  host: myhost
  user: springboot
  dbname: springboot
  secretName: secretName
  secretKey: password
  secretKeyroot: rootpassword
  url: jdbc:mysql://springboot-mysql/springboot?autoReconnect=true
  driver: com.mysql.jdbc.Driver
```

{{% alert title="Warning" color="secondary" %}}
Make sure the `url` contains the correct service name. The service name is based on the chart name `{{ include "mychart.fullname" . }}-mysql` (see the service template above).
{{% /alert %}}


To upgrade your existing release run:

```bash
helm upgrade myapp --namespace [USER] ./mychart
```