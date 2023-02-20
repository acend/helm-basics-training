---
title: "7.2 Template the database backend"
weight: 72
sectionnumber: 7.2
---

In this lab we are going to templates the resources that are necessary to deploy a MariaDB database as a backend to our `example-web-python` application. Before we start creating those templates we want to have a look at a couple of best practices.


## Resource names

When looking at the app templates of our `mychart` chart, the name of the resource is always defined by a helper function:

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
There are two helper functions you can use to generate these labels

* `{{- include "helm-complex-chart.selectorLabels" . }}` to generate the so-called selector labels. Primary you can use this function in Service templates under `.spec.selector` 
* `{{- include "helm-complex-chart.labels" . }}` to generate the Helm common labels. Mainly used for the `.metadata.labels` in any resource template. 

## Task {{% param sectionnumber %}}.1: Add a new helper function

As mentioned above, the `_helpers.tpl` file provides some neat functions to generate the Labels and Selectors for Kubernetes resources.
But if we take a closer look, we see that we can not simply use the same labels and selectors. Because we have two different deployments, each with its own service, we need to ensure that the selecttor and labels ardistinct for both deployments and services.

Let's open the `_helpers.tpl` file and scroll down to the helper function for the labels.

```yaml

{{/*
Common labels
*/}}
{{- define "helm-complex-chart.labels" -}}
helm.sh/chart: {{ include "helm-complex-chart.chart" . }}
{{ include "helm-complex-chart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "helm-complex-chart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "helm-complex-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

```

Now we're going to add the same functions to generate the labels for the MariaDB resources.
Add following content to the helpers file, so that we have two new helper functions.

* `helm-complex-chart.labelsDatabase` to generate the common labels for MariaDB
* `helm-complex-chart.selectorLabelsDatabase` to generate the service selector labels for MariaDB


```yaml

{{/*
Common labels for MaraiDB
*/}}
{{- define "helm-complex-chart.labelsDatabase" -}}
helm.sh/chart: {{ include "helm-complex-chart.chart" . }}
{{ include "helm-complex-chart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels for MariaDB
*/}}
{{- define "helm-complex-chart.selectorLabelsDatabase" -}}
app.kubernetes.io/name: {{ include "helm-complex-chart.name" . }}-mariadb
app.kubernetes.io/instance: {{ .Release.Name }}-mariadb
{{- end }}

```

## Task {{% param sectionnumber %}}.2: Edit the MariaDB template files

We want to add a MariaDB database and use it as a backend for our `example-web-python` application. Using the following Kubernetes resource file, let's take a look at the template files for the MariaDB database:

There are three template files which are neccessary for the MariaDB deployment.
* `deployment-mariadb.yaml`
* `service-mariadb.yaml`
* `secret-mariadb.yaml`


Let's examine the MariaDB Deployment template first. As we can see, the deployment of the MariaDB is very similar to the App Deployment. 

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

Furthermore you have already a Secret which contains the passwords for the Database


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

To connect to the database, we also have a service:

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
    {{- include "helm-complex-chart.labelsDatabase" . | nindent 4 }}
spec:
  replicas: 1
  selector:
    matchLabels:
      {{- include "helm-complex-chart.selectorLabelsDatabase" . | nindent 6 }}
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        {{- include "helm-complex-chart.labelsDatabase" . | nindent 8 }}
    spec:
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
      volumes:
      - name: mariadb-persistent-storage
        emptyDir: {}
```

The secret `templates/secret-mariadb.yaml` file should look like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-mariadb
  labels:
    {{- include "helm-complex-chart.labelsDatabase" . | nindent 4 }}
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
    {{- include "helm-complex-chart.labelsDatabase" . | nindent 4 }}
spec:
  ports:
    - port: 3306
      targetPort: mariadb
      protocol: TCP
      name: mariadb
  selector:
    {{- include "helm-complex-chart.selectorLabelsDatabase" . | nindent 4 }}
  clusterIP: None
```

Then we need to add the following to our `values.yaml`:

```yaml
database:
  enabled: true
  databaseuser: acend
  databasename: acenddb
  databasepassword: mysuperpassword123
  databaserootpassword: mysuperrootpassword123
  image:
    repository: registry.puzzle.ch/docker.io/mariadb
    pullPolicy: IfNotPresent
    tag: "10.5"
```


Finally, to upgrade the existing release run:

```bash
helm upgrade myapp ./mychart --namespace <namespace>
```

{{% alert title="Note" color="primary" %}}
Remember the `--dry-run` option from lab 2. This allows you to render the templates without applying them to the cluster.
{{% /alert %}}



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

Copy the test file `templates/tests/test-connection.yaml` to `templates/tests/test-connection-database.yaml` from the previous section and start to modify it.

{{% onlyWhen openshift %}}
But first we need to adjust the existing `test-connection.yaml` Test. Add the additional argument `--spider` to make the test work within an OpenShift environment.

{{< highlight YAML "hl_lines=14" >}}
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mychart.fullname" . }}-test-connection"
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['--spider', '{{ include "mychart.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
{{< /highlight >}}
{{% /onlyWhen %}}


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
LAST DEPLOYED: Tue Nov  8 20:22:59 2022
NAMESPACE: user1
STATUS: deployed
REVISION: 5
TEST SUITE:     myapp-mychart-test-connection
Last Started:   Tue Nov  8 20:23:04 2022
Last Completed: Tue Nov  8 20:23:09 2022
Phase:          Succeeded
TEST SUITE:     myapp-mychart-test-connection
Last Started:   Tue Nov  8 20:23:09 2022
Last Completed: Tue Nov  8 20:23:15 2022
Phase:          Succeeded
```


## Task {{% param sectionnumber %}}.5: Cleanup

If you're happy with the result, clean up your namespace:

```bash
helm uninstall myapp --namespace <namespace>
```
