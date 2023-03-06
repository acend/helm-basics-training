---
title: "6.2 Template the database backend"
weight: 62
sectionnumber: 6.2
---

In this lab we are going to templates the resources that are necessary to deploy a MariaDB database as a backend to our `example-web-python` application. Before we start creating those templates we want to have a look at a couple of best practices.


## Resource names


When looking at the app templates of our `mychart` chart, the name of the resource is always defined by a helper function:

```yaml
name: {{ include "helm-complex-chart.fullname" . }}
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

But first add the new values for the ...

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

{{% alert title="Note" color="info" %}}
Remember the `--dry-run` option from lab 2. This allows you to render the templates without applying them to the cluster.
{{% /alert %}}


### Deployment


Let's examine the MariaDB Deployment template first. As we can see, the deployment of the MariaDB is very similar to the App Deployment.
Make following changes to create a reusable template


* Replace `metadata.labels` with the template function `helm-complex-chart.labelsDatabase` we created before
* Replace `spec.selector.matchLabels` with the template function we created before
* Replace `spec.template.metadata.labels` with the template function `helm-complex-chart.labelsDatabase` we created before
* Replace `spec.template.spec.containers[0].image` with  
* Replace `spec.template.spec.containers[0].imagePullPolicy` with
* Replace under `spec.template.spec.containers[0].env` the values from following keys
  * `MYSQL_USER` value with `.Values.database.databaseuser`
  * `MYSQL_DATABASE` value with `.Values.database.databaseuser`

{{< highlight YAML "hl_lines= 7-11 16-17 23-24 27-29 34 46" >}}
{{- if .Values.database.enabled }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-mariadb
  labels:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: myapp
    helm.sh/chart: helm-complex-chart-0.1.0
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
              name: {{ .Release.Name }}-mariadb
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              key: mariadb-root-password
              name: {{ .Release.Name }}-mariadb
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
{{- end }}
{{< /highlight >}}


### Secret


Furthermore you have already a Secret `mariadb-secret.yaml` which contains the passwords for the Database
Make following changes to create a reusable template

* Replace `metadata.labels` with the template function `helm-complex-chart.labelsDatabase` we created before
* Replace under `data` the values from following keys
  * `mariadb-root-password` with **base64 encoded** `.Values.database.databaseuser` value
  * `mariadb-password` with **base64 encoded** `.Values.database.databaseuser` value


{{< highlight YAML "hl_lines= 7-11 13-14" >}}
{{- if .Values.database.enabled }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-mariadb
  labels:
    app.kubernetes.io/name: mariadb
    app.kubernetes.io/instance: myapp
    helm.sh/chart: helm-complex-chart-0.1.0
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
data:
  mariadb-root-password: "bXlzdXBlcnJvb3RwYXNzd29yZDEyMw=="
  mariadb-password: "bXlzdXBlcnBhc3N3b3JkMTIz"
{{- end }}
{{< /highlight >}}

When creating the template files, make sure that a user can specify the `mariadb-password` and `mariadb-root-password` from the secret using a variable.


### Service


To connect to the database, we also have a service
Make the following changes to create a reusable template

* Replace `metadata.name` with
* Replace `metadata.labels` with the template function we created before
* Replace `metadata.spec.selector` with the template function we created before

{{< highlight YAML "hl_lines=4 6-8 16-17" >}}
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
{{< /highlight >}}

When your changes are ready, upgrade the already deployed release with the new version.


## {{% param sectionnumber %}}.2: Solution


The template file for the MariaDB database `templates/deployment-mariadb.yaml` could look like this.

The following points need to be taken into consideration when creating the template:

* The helper `helm-complex-chart.fullname` will return `release-helm-complex-chart`. Since our first deployment for the `example-web-python` application already uses this name, we have to choose a name for the mariadb instead.
  * Let's take `<releasename>-mariadb` instead. As an alternative, we could also alter the full name helper to accept an additional name, which would be different for each deployment.
* The same applies to the label `app.kubernetes.io/name`. We can't therefore use the included `helm-complex-chart.labels`. We could also alter the helper function or in our case simply just add the labels directly.
* In the deployment templates we reference our secrets by again using the full name `<releasename>-mariadb`.


### Deployment


```yaml
{{- if .Values.database.enabled }}
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
        {{- include "helm-complex-chart.selectorLabelsDatabase" . | nindent 8 }}
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
{{- end }}
```


### Secret


The secret `templates/mariadb-secret.yaml` file should look like this:

```yaml
{{- if .Values.database.enabled }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Release.Name }}-mariadb
  labels:
    {{- include "helm-complex-chart.labelsDatabase" . | nindent 4 }}
data:
  mariadb-password: {{ .Values.database.databasepassword | b64enc }}
  mariadb-root-password: {{ .Values.database.databaserootpassword | b64enc }}
{{- end }}
```

Note the `| b64enc`, which is a built-in function to encode strings with base64.


### Service


The service at `templates/mariadb-service.yaml` for our MySQL database should look similar to this:

```yaml
{{- if .Values.database.enabled }}
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
{{- end }}
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


Finally, to install the release run:

```bash
helm upgrade -i myapp . --namespace $USER
```


## Task {{% param sectionnumber %}}.5: Cleanup

If you're happy with the result, clean up your namespace:

```bash
helm uninstall myapp --namespace $USER
```
