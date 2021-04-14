---
title: "4.2 A new backend"
weight: 42
sectionnumber: 4.2
---

In this lab we are going to create the templates that are necessary to deploy a MariaDB database as a backend to our `example-web-python` application. Before we start creating those templates we want to have a look at a couple of best practices.


## Resource names

When looking at the templates of our `mychart` chart, the name of the resource is always defined by a helper function:

```yaml
name: {{ include "mychart.fullname" . }}
```

Resources within the same Namespace and of the same resource type must have unique names. Since a Helm chart can be instantiated multiple times in different releases, it's best to include the release name as part of the resource name:

`<releasename>-<chartname>`

Have a look at the helper function in `mychart/templates/_helpers.tpl` for more details and keep this concept in mind when creating new templates.


## Labels

There are a couple of [recommended Kubernetes labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/){{% onlyWhen openshift %}} (that also apply to OpenShift){{% /onlyWhen %}}, which help to make interoperability between different tools and concepts easier and define a kind of standard.

We use the following two labels as so-called selector labels. These are labels that are used on services to define which pods belong to them:

* `app.kubernetes.io/name`: Resource name
* `app.kubernetes.io/instance`: Release name

And a couple of additional ones to label our resources with supplemental information:

* `helm.sh/chart`: Chart name, e.g. `mychart-0.1.0`
* `app.kubernetes.io/version`: Chart version, e.g. `1.16.0`
* `app.kubernetes.io/managed-by: Helm`

Similar to the resource name we can also use helpers to generate the necessary labels.


## Task {{% param sectionnumber %}}.1: Create template files

We want to add a MariaDB database and use it as a backend for our `example-web-python` application. Using the following Kubernetes resource file, create a new template file for the MariaDB database:

{{% alert title="Note" color="primary" %}}
You can use the existing templates (`deployment.yaml`, `service.yaml`) as a starting point to create the new templates.
{{% /alert %}}

{{% onlyWhenNot mobi %}}

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myfirstrelease-mychart-mariadb
  labels:
    app.kubernetes.io/name: mychart-mariadb
    app.kubernetes.io/instance: myfirstrelease
    # additional Labels ...
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: mychart-mariadb
      app.kubernetes.io/instance: myfirstrelease
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mychart-mariadb
        app.kubernetes.io/instance: myfirstrelease
    spec:
      containers:
      - image: mariadb:10.5
        name: mariadb
        args:
          - "--ignore-db-dir=lost+found"
        env:
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: database-user
              name: mariadb
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-password
              name: mariadb
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-root-password
              name: mariadb
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database-name
              name: mariadb
        livenessProbe:
          tcpSocket:
            port: 3306
        ports:
        - containerPort: 3306
          name: mariadb
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mariadb-persistent-storage
        emptyDir: {}
```

{{% /onlyWhenNot %}}

{{% onlyWhen mobi %}}
You have to use `docker-registry.mobicorp.ch/puzzle/k8s/kurs/mariadb:10.5` as the container image.

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myfirstrelease-mychart-mariadb
  labels:
    app.kubernetes.io/name: mychart-mariadb
    app.kubernetes.io/instance: myfirstrelease
    # additional Labels ...
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: mychart-mariadb
      app.kubernetes.io/instance: myfirstrelease
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mychart-mariadb
        app.kubernetes.io/instance: myfirstrelease
    spec:
      containers:
      - image: docker-registry.mobicorp.ch/puzzle/k8s/kurs/mariadb:10.5
        name: mariadb
        args:
          - "--ignore-db-dir=lost+found"
        env:
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: database-user
              name: myfirstrelease-mychart-mariadb
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-password
              name: myfirstrelease-mychart-mariadb
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-root-password
              name: myfirstrelease-mychart-mariadb
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database-name
              name: myfirstrelease-mychart-mariadb
        livenessProbe:
          tcpSocket:
            port: 3306
        ports:
        - containerPort: 3306
          name: mariadb
          protocol: TCP
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mariadb-persistent-storage
        emptyDir: {}
```

{{% /onlyWhen %}}

You also have to create a template for the secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myfirstrelease-mychart-mariadb
  labels:
    app.kubernetes.io/name: mychart-mariadb
    app.kubernetes.io/instance: myfirstrelease
    # additional Labels ...
data:
  database-name: YWNlbmRkYg==
  database-password: bXlzdXBlcnBhc3N3b3JkMTIz
  database-root-password: bXlzdXBlcnJvb3RwYXNzd29yZDEyMw==
  database-user: YWNlbmQ=
```

When creating the template files, make sure that a user can specify the `database-name`, `database-password`, `database-root-password` and `database-user` from the secret using a variable.

To connect to the database, we also need a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myfirstrelease-mychart-mariadb
  labels:
    app.kubernetes.io/name: mychart-mariadb
    app.kubernetes.io/instance: myfirstrelease
    # additional Labels ...
spec:
  ports:
    - port: 3306
      targetPort: mariadb
      protocol: TCP
      name: mariadb
  selector:
    app.kubernetes.io/name: mychart-mariadb
    app.kubernetes.io/instance: myfirstrelease
  clusterIP: None
```

Have a look at the already existing file `service.yaml` and use this as a starting point for your `mariadb` service.

When your changes are ready, upgrade the already deployed release with the new version.


### Solution

The template file for the MariaDB database `templates/deployment-mariadb.yaml` could look like this.

The following points need to be taken into consideration when creating the template:

* The helper `mychart.fullname` will return `release-mychart`. Since our first deployment for the `example-web-python` application already uses this name, we use `-mariadb` as a postfix. As an alternative we could also alter the fullname helper to accept an additional name, which would be different for each deployment.
* The same applies to the label `app.kubernetes.io/name`. We can't therefore use the included `mychart.labels`. We could also alter the helper function or in our case simply just add the labels directly.
* In the deployment templates we reference our secrets by again using the full name `{{ include "mychart.fullname" . }}-mariadb`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}-mariadb
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}-mariadb
    app.kubernetes.io/instance: {{ .Release.Name }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "mychart.name" . }}-mariadb
      app.kubernetes.io/instance: {{ .Release.Name }}
  strategy:
    type: Recreate
  template:
    metadata:
      {{- with .Values.database.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      labels:
        app.kubernetes.io/name: {{ include "mychart.name" . }}-mariadb
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      {{- with .Values.database.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "mychart.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.database.podSecurityContext | nindent 8 }}
      containers:
      - image: "{{ .Values.database.image.repository }}:{{ .Values.database.image.tag}}"
        name: mariadb
        imagePullPolicy: {{ .Values.database.image.pullPolicy }}
        args:
          - "--ignore-db-dir=lost+found"
        env:
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: database-user
              name: {{ include "mychart.fullname" . }}-mariadb
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-password
              name: {{ include "mychart.fullname" . }}-mariadb
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: database-root-password
              name: {{ include "mychart.fullname" . }}-mariadb
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database-name
              name: {{ include "mychart.fullname" . }}-mariadb
        livenessProbe:
          tcpSocket:
            port: 3306
        ports:
        - containerPort: 3306
          name: mariadb
        volumeMounts:
        - name: mariadb-persistent-storage
          mountPath: /var/lib/mysql
        resources:
          {{- toYaml .Values.database.resources | nindent 12 }}
      volumes:
      - name: mariadb-persistent-storage
        emptyDir: {}
      {{- with .Values.database.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.database.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.database.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

The secret `templates/secret-mariadb.yaml` file should look like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "mychart.fullname" . }}-mariadb
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}-mariadb
    app.kubernetes.io/instance: {{ .Release.Name }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
data:
  database-name: {{ .Values.database.databasename | b64enc }}
  database-password: {{ .Values.database.databasepassword | b64enc }}
  database-root-password: {{ .Values.database.databaserootpassword | b64enc }}
  database-user: {{ .Values.database.databaseuser | b64enc }}
```

Note the `| b64enc`, which is a built-in function to encode strings with base64.

The service at `templates/service-mariadb.yaml` for our MySQL database should look similar to this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}-mariadb
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}-mariadb
    app.kubernetes.io/instance: {{ .Release.Name }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  ports:
    - port: 3306
      targetPort: mariadb
      protocol: TCP
      name: mariadb
  selector:
    app.kubernetes.io/name: {{ include "mychart.name" . }}-mariadb
    app.kubernetes.io/instance: {{ .Release.Name }}
  clusterIP: None
```

Then we need to add the following to our `values.yaml`:
{{% onlyWhenNot mobi %}}

```yaml
database:
  databaseuser: acend
  databasename: acenddb
  databasepassword: mysuperpassword123
  databaserootpassword: mysuperrootpassword123
  image:
    repository: mariadb
    pullPolicy: IfNotPresent
    tag: "10.5"
  imagePullSecrets: []
  podAnnotations: {}
  podSecurityContext: {}
  resources: {}
  nodeSelector: {}
  tolerations: []
  affinity: {}
```

{{% /onlyWhenNot %}}

{{% onlyWhen mobi %}}

```yaml
database:
  databaseuser: acend
  databasename: acenddb
  databasepassword: mysuperpassword123
  databaserootpassword: mysuperrootpassword123
  image:
    repository: docker-registry.mobicorp.ch/puzzle/k8s/kurs/mariadb
    pullPolicy: IfNotPresent
    tag: "10.5"
  imagePullSecrets: []
  podAnnotations: {}
  podSecurityContext: {}
  resources: {}
  nodeSelector: {}
  tolerations: []
  affinity: {}
```

{{% /onlyWhen %}}

Finally, to upgrade the existing release run:

```bash
helm upgrade myapp ./mychart --namespace <namespace>
```


## Task {{% param sectionnumber %}}.2: Connect to the database

In order for the python application to be able to connect to the newly deployed database, we need to add some environment variables to our deployment. The goal of this task is to allow a user to set them via values.

Add the following environment variables:

* `MYSQL_DATABASE_NAME` value from the secret you created in task 1
* `MYSQL_DATABASE_PASSWORD` value from the secret you created in task 1
* `MYSQL_DATABASE_ROOT_PASSWORD` value from the secret you created in task 1
* `MYSQL_DATABASE_USER` value from the secret you created in task 1
* `MYSQL_URI` with the value `mysql://$(MYSQL_DATABASE_USER):$(MYSQL_DATABASE_PASSWORD)@<servicename of mariadb>/$(MYSQL_DATABASE_NAME)`


### Solution Task {{% param sectionnumber %}}.2

Change your `templates/deployment.yaml` and include the new environment variables:

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
          env:
          - name: MYSQL_DATABASE_NAME
            valueFrom:
              secretKeyRef:
                key: database-name
                name: {{ include "mychart.fullname" . }}-mariadb
          - name: MYSQL_DATABASE_PASSWORD
            valueFrom:
              secretKeyRef:
                key: database-password
                name: {{ include "mychart.fullname" . }}-mariadb
          - name: MYSQL_DATABASE_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                key: database-root-password
                name: {{ include "mychart.fullname" . }}-mariadb
          - name: MYSQL_DATABASE_USER
            valueFrom:
              secretKeyRef:
                key: database-user
                name: {{ include "mychart.fullname" . }}-mariadb
          - name: MYSQL_URI
            value: mysql://$(MYSQL_DATABASE_USER):$(MYSQL_DATABASE_PASSWORD)@{{ include "mychart.fullname" . }}-mariadb/$(MYSQL_DATABASE_NAME)
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

There should not be any changes in the `values.yaml`:

```yaml
database:
  databaseuser: acend
  databasename: acenddb
  databasepassword: mysuperpassword123
  databaserootpassword: mysuperrootpassword123
  image:
    repository: mariadb
    pullPolicy: IfNotPresent
    tag: "10.5"
  imagePullSecrets: []
  podAnnotations: {}
  podSecurityContext: {}
  resources: {}
  nodeSelector: {}
  tolerations: []
  affinity: {}
```

To upgrade your existing release run:

```bash
helm upgrade myapp ./mychart --namespace <namespace>
```


## Task {{% param sectionnumber %}}.3: Check

Check whether the attachment of the new backend worked by either looking at the Pod's logs: In there the application tells you which backend it uses, this should of course be the database. Or simply access the application in your browser, create an entry, re-deploy the application Pod (e.g. by scaling it down and up again) and check if your entry is still there.


## Task {{% param sectionnumber %}}.4: Cleanup

If you're happy with the result, clean up your namespace:

```bash
helm uninstall myapp --namespace <namespace>
```
