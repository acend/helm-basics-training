---
title: "4.2 - Add a database to your app"
weight: 42
---

### Task 1: Create Template Files for MySQL Deployment

We want to add a MySQL database and us it as a backend for our `example-spring-boot` application. Using the following Kubernetes resource file for a MySQL database, create a new template file for the MySQL database:

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

You also have to create a template for the `mysql-root-password` Secret:

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

To connect to the database, you also need a Service:

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

Have a look at the already existing file `service.yaml` and use this as a help for your `springboot-mysl` Service.

When your changes are ready, upgrade your already deployed release this the new version.


{{% collapse solution-1 "Solution Task 1" %}}

The template file for the MySQL database `templates/deploy-mysql.yaml` could look like this:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}-mysql
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
    tier: mysql
spec:
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
        tier: mysql
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

Notice the `| b64enc`, which is a builtin function to encode string with base64


The Service for your MySQL database should look like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}-mysql
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
    tier: mysql
spec:
  type: ClusterIP
  ports:
    - port: 3306
      protocol: TCP
  selector:
    {{- include "mychart.selectorLabels" . | nindent 4 }}
    tier: mysql

```

and then in your `values.yaml` you need to add:

```yaml
database:
  host: myhost
  user: springboot
  dbname: sprintboot
  secretName: mysql-root-password
  secretKey: password
  secretKeyroot: rootpassword
  password: mysuperpassword123
  rootpassword: mysuperrootpassword123
```

To upgrade your existing release run: 

```bash
helm upgrade myreleasename .
```


{{% /collapse %}}



### Task 2: Add Environment Variables to use the MySQL Database in your App

We need some environment variables in our Deployment. The goal of this Task is to allow a user to set them via values.

Add the following environment variables:

* `SPRING_DATASOURCE_USERNAME` with value `springboot`
* `SPRING_DATASOURCE_PASSWORD` value from secret you created in task 1
* `SPRING_DATASOURCE_DRIVER_CLASS_NAME` with value `com.mysql.jdbc.Driver`
* `SPRING_DATASOURCE_URL` with value `jdbc:mysql://springboot-mysql/springboot?autoReconnect=true`


{{% collapse solution-2 "Solution Task 2" %}}

Change your `template/deployment.yml` and include the new environment variables:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "test.fullname" . }}
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
              path: /health
              port: http
          readinessProbe:
            httpGet:
              path: /health
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

and your `values.yaml` should contain now (including the previously defined values from task 2)

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

{{% notice tip %}}

Make your the `url` contains the correct Service Name. The Service Name is build based on the chartname `{{ include "mychart.fullname" . }}-mysql` (see template for the Service aboce)

{{% /notice %}}

To upgrade your existing release run: 

```bash
helm upgrade myreleasename .
```

{{% /collapse %}}

