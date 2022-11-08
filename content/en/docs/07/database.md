---
title: "7.2 A new backend"
weight: 72
sectionnumber: 7.2
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
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-mariadb
  labels:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: myapp
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: mariadb
      app.kubernetes.io/instance: myapp
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mariadb
        app.kubernetes.io/instance: myapp
    spec:
      containers:
      - image: "registry.puzzle.ch/docker.io/mariadb:10.5"
        name: mariadb
        imagePullPolicy: IfNotPresent
        args:
          - "--ignore-db-dir=lost+found"
        env:
        - name: MYSQL_USER
          value: "acend"
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: mariadb-password
              name: myapp-mariadb
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: mariadb-root-password
              name: myapp-mariadb
        - name: MYSQL_DATABASE
          value: "acenddb"
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
You have to use `<registry-url>/puzzle/k8s/kurs/mariadb:10.5` as the container image.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-mariadb
  labels:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: myapp
    # additional Labels ...
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: mariadb
      app.kubernetes.io/instance: myapp
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mariadb
        app.kubernetes.io/instance: myapp
    spec:
      containers:
      - image: <registry-url>/puzzle/k8s/kurs/mariadb:10.5
        name: mariadb
        args:
          - "--ignore-db-dir=lost+found"
        env:
        - name: MYSQL_USER
          value: "acend"
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: mariadb-password
              name: myapp-mariadb
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: mariadb-root-password
              name: myapp-mariadb
        - name: MYSQL_DATABASE
          value: "acenddb"
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
  name: myapp-mariadb
  labels:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: myapp
    # additional Labels ...
data:
  mariadb-root-password: "bXlzdXBlcnJvb3RwYXNzd29yZDEyMw=="
  mariadb-password: "bXlzdXBlcnBhc3N3b3JkMTIz"
```

When creating the template files, make sure that a user can specify the `mariadb-password` and `mariadb-root-password` from the secret using a variable.

To connect to the database, we also need a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-mariadb
  labels:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: myapp
    # additional Labels ...
spec:
  ports:
    - port: 3306
      targetPort: mariadb
      protocol: TCP
      name: mariadb
  selector:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: myapp
  clusterIP: None
```

Have a look at the already existing file `service.yaml` and use this as a starting point for your `mariadb` service.

When your changes are ready, upgrade the already deployed release with the new version.


### Solution

The template file for the MariaDB database `templates/deployment-mariadb.yaml` could look like this.

The following points need to be taken into consideration when creating the template:

* The helper `mychart.fullname` will return `release-mychart`. Since our first deployment for the `example-web-python` application already uses this name, we have to choose a name for the mariadb instead.
  * Let's take `<releasename>-mariadb` instead. As an alternative we could also alter the fullname helper to accept an additional name, which would be different for each deployment.
* The same applies to the label `app.kubernetes.io/name`. We can't therefore use the included `mychart.labels`. We could also alter the helper function or in our case simply just add the labels directly.
* In the deployment templates we reference our secrets by again using the full name `<releasename>-mariadb`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-mariadb
  labels:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: {{ .Release.Name }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: mariadb
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
        app.kubernetes.io/name: mariadb
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
          value: {{ .Values.database.databaseuser }}
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: mariadb-password
              name: {{ .Release.Name }}-mariadb
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: mariadb-root-password
              name: {{ .Release.Name }}-mariadb
        - name: MYSQL_DATABASE
          value: {{ .Values.database.databasename }}
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
  name: {{ .Release.Name }}-mariadb
  labels:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: {{ .Release.Name }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
data:
  mariadb-password: {{ .Values.database.databasepassword | b64enc }}
  mariadb-root-password: {{ .Values.database.databaserootpassword | b64enc }}
```

Note the `| b64enc`, which is a built-in function to encode strings with base64.

The service at `templates/service-mariadb.yaml` for our MySQL database should look similar to this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-mariadb
  labels:
    app.kubernetes.io/name: mariadb
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
    app.kubernetes.io/name: mariadb
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
    repository: registry.puzzle.ch/docker.io/mariadb
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
    repository: <registry-url>/puzzle/k8s/kurs/mariadb
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

{{% alert title="Note" color="primary" %}}
Remember the `--dry-run` option from lab 2. This allows you to render the templates without applying them to the cluster.
{{% /alert %}}


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


### Solution Task {{% param sectionnumber %}}.3

First we need to exectute following command to determine the pod name
```bash
{{% param cliToolName %}} get pods -n <namespace>
```

{{< highlight bash "hl_lines=3" >}}
NAME                                    READY   STATUS      RESTARTS   AGE
myapp-mychart-test-connection          0/1     Completed   0          22m
myapp-mychart-7cc85f99db-n4lsc          1/1     Running     0          12m
myapp-mychart-mariadb-74ddcc878-268ts   1/1     Running     0          12m
webshell-67f4cf8c59-st4rg               2/2     Running     0          2d22h
{{< /highlight >}}

Next execute following command to show the logs
```bash
{{% param cliToolName %}} logs myapp-mychart-7cc85f99db-n4lsc
```


As you can see in the log output, our application is now connected to the fresh deployed database
> Never log sensitive informations like database connection strings which contain the password in plain text!

```bash
Using DB:  mysql://acend:mysuperpassword123@myapp-mychart-mariadb/acenddb
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


## Task {{% param sectionnumber %}}.4: Add a test to your chart

As we learned in the previous section, Helm gives us the availability to run automated test during the Helm deployment.
Now it's time to write our first test.
The test should meet following requirements:

* Image: mariadb
* Env Variables:
  * `MYSQL_DATABASE_HOST` from secret
  * `MYSQL_DATABASE_USER` from secret
  * `MYSQL_DATABASE_PASSWORD` from secret
* Command: `mysql --host=$MYSQL_DATABASE_HOST --user=$MYSQL_DATABASE_USER --password=$MYSQL_DATABASE_PASSWORD`
* Annotation: `helm.sh/hook: test`

You can copy the test file `templates/tests/test-connection.yaml` to `templates/tests/test-connection-database.yaml` from the previous section and start to modify it.


### Solution Task {{% param sectionnumber %}}.4

```yaml
{{- $fullName := include "mychart.fullname" . -}}
{{- $all := . -}}
---
apiVersion: v1
kind: Pod
metadata:
  name: "{{ $fullName }}-test-connection"
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}-mariadb-test
    app.kubernetes.io/instance: {{ .Release.Name }}-test
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - image: "{{ .Values.database.image.repository }}:{{ .Values.database.image.tag}}"
      name: mariadb
      imagePullPolicy: {{ .Values.database.image.pullPolicy }}
      command: ['mysql']
      args: ['--host=$(MYSQL_DATABASE_HOST)', '--user=$(MYSQL_DATABASE_USER)', '--password=$(MYSQL_DATABASE_PASSWORD)']
      env:
      - name: MYSQL_DATABASE_HOST
        value: {{ .Release.Name }}-mariadb
      - name: MYSQL_DATABASE_PASSWORD
        valueFrom:
          secretKeyRef:
            key: mariadb-password
            name: {{ .Release.Name }}-mariadb
      - name: MYSQL_DATABASE_USER
        value: {{ .Values.database.databaseuser }}
  restartPolicy: Never
```

To upgrade your existing release run:

```bash
helm upgrade myapp ./mychart --namespace <namespace>
```

To run the test manually:

```bash
helm test myapp --namespace <namespace>
```

Then you should see following output

```bash
NAME: myapp
LAST DEPLOYED: Sun Oct 10 15:46:33 2021
NAMESPACE: student3
STATUS: deployed
REVISION: 4
TEST SUITE:     myapp-mychart--test-connection
Last Started:   Sun Oct 10 15:46:44 2021
Last Completed: Sun Oct 10 15:46:46 2021
Phase:          Succeeded
TEST SUITE:     myapp-mychart-test-connection
Last Started:   Sun Oct 10 15:46:46 2021
```


## Task {{% param sectionnumber %}}.5: Cleanup

If you're happy with the result, clean up your namespace:

```bash
helm uninstall myapp --namespace <namespace>
```
