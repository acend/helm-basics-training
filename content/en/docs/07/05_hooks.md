---
title: "7.5 Helm hooks"
weight: 75
sectionnumber: 7.5
---

In the previous lab we learned how to deploy our python example application and connect it to a MariaDB. In this chapter we are going to look at a mechanism of Helm called hooks, which let's chart developers intervene / interact at certain points in a release's life cycle.


## Task {{% param sectionnumber %}}.1: Chart Hooks

At some point when developing helm charts we would like to interact or alter certain behaviour of our deployed resources. For example in this usecase we would like to have test data available in our database when releasing our chart. With the hook functionality of helm we can alter the life cycle and intervene at certain points. Hooks are regular templates specially annotated that causes Helm to utilize them differently. The tests used in the previous chapter are a special case of helm hooks. The following hooks are available for you to alter the behaviour of the chart:

* pre-install: Executes after templates are rendered, but before any resources are created in Kubernetes
* post-install: Executes after all resources are loaded into Kubernetes
* pre-delete: Executes on a deletion request before any resources are deleted from Kubernetes
* post-delete: Executes on a deletion request after all of the release's resources have been deleted
* pre-upgrade: Executes on an upgrade request after templates are rendered, but before any resources are updated
* post-upgrade: Executes on an upgrade request after all resources have been upgraded
* pre-rollback: Executes on a rollback request after templates are rendered, but before any resources are rolled back
* post-rollback: Executes on a rollback request after all resources have been modified
* test: Executes when the Helm test subcommand is invoked

Hooks are controlled and configured via kubernetes annotations. For example to configure the life cycle the hook should be hooked into we use the annotation:

```yaml
[...]
  annotations:
    "helm.sh/hook": post-install
[...]
```

The resource can even implement multiple hooks:

```yaml

[...]
  annotations:
    "helm.sh/hook": post-install, post-upgrade
[...]

```

When working with multiple hooks hooking into the same life cycle stage, we can alter the execution or runtime behaviour by giving them different weights with the annotation:

```yaml

[...]
  annotations:
    "helm.sh/hook-weight": "-5"
[...]

```

Hook weights can be positive or negative numbers but must be represented as strings. When Helm starts the execution cycle of hooks of a particular Kind it will sort those hooks in ascending order.


## Task {{% param sectionnumber %}}.2: Example hook

In this example we would like to populate our database with some test data before finishing our install / upgrade life cycle. We can use a helm hook to inject our logic after installing or upgrading our helm release. Create the following hook `templates/hook.yaml`:

```yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-post-install-hook"
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      name: "{{ .Release.Name }}"
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
        helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    spec:
      restartPolicy: Never
      containers:
      - name: post-install-job
        image: "alpine:3.3"
        command: ["/bin/sleep","{{ default "10" .Values.sleepyTime }}"]

```

This basic hook creates a kubernetes Job which sleeps for <.Value.sleepyTime> or 10 seconds after the installation life cycle of your helm chart.

Notice the `metadata.annotations` block which tells Helm how the hook should interact during the release's life cycle:

```yaml

[...]
  annotations:
    # This is what defines this resource as a hook. Without this line, the
    # job is considered part of the release.
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
[...]

```

Let's install the release and check whether the hook was deployed.
First open a second terminal and run the following command:

```bash
{{% param cliToolName %}} get pod -w --namespace <namespace>
```

```bash
helm upgrade -i myapp ./mychart --namespace <namespace>
```

The output of the `{{% param cliToolName %}} get pod` command should show the deployment of the `post-install-hook` pod.


## Task {{% param sectionnumber %}}.3: Write your own hook

It's time to get our hands dirty! Write two hooks to alter the release's life cycle:

* Create a post-install/post-upgrade hook to create a database table `test` and populate it with data of your favor. In the example solution we create a table `test` with two fields: An autoincrement index `id` and a char field called `name` to store some strings.  Ensure this job is running after the job below.
* Create a post-install/post-upgrade hook to drop the table `test` if it exists to have a clean state before releasing. Ensure this job is running before the job above.

To interact with the MariaDB deployed you can use the image `registry.puzzle.ch/docker.io/mariadb:10.5` for example. You can interact and access the database with `$ mysql --host=<hostname> --user=<username> --password=<password> --database=<database> -e <sqlCommand>`.


{{% details title="Solution" %}}

First Post-install / -upgrade hook:

```yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-hook1"
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
  annotations:
    "helm.sh/hook": post-install, post-upgrade
    "helm.sh/hook-weight": "-5"
spec:
  template:
    metadata:
      name: {{ .Release.Name | quote }}
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
        helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    spec:
      restartPolicy: Never
      containers:
        - name: post-install-job
          image: "{{ .Values.database.image.repository }}:{{ .Values.database.image.tag}}"
          imagePullPolicy: {{ .Values.database.image.pullPolicy }}
          command: ['/bin/bash', '-c']
          args:
          - >
            mysql --host=$(MYSQL_DATABASE_HOST) --user=$(MYSQL_DATABASE_USER) --password=$(MYSQL_DATABASE_PASSWORD) --database=$(MYSQL_DATABASE) -e "DROP TABLE IF EXISTS test;"
          env:
          - name: MYSQL_DATABASE_USER
            value: {{ .Values.mariadb.auth.username }}
          - name: MYSQL_DATABASE_PASSWORD
            value: {{ .Values.mariadb.auth.password }}
          - name: MYSQL_DATABASE_ROOT_PASSWORD
            value: {{ .Values.mariadb.auth.rootPassword }}
          - name: MYSQL_DATABASE
            value: {{ .Values.mariadb.auth.database }}
          - name: MYSQL_DATABASE_HOST
            value: {{ .Release.Name }}-mariadb

```

Second Post-install / -upgrade hook:

```yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-hook2"
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
  annotations:
    "helm.sh/hook": post-install, post-upgrade
    "helm.sh/hook-weight": "10"
spec:
  template:
    metadata:
      name: {{ .Release.Name | quote }}
      labels:
        app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
        app.kubernetes.io/instance: {{ .Release.Name | quote }}
        helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    spec:
      restartPolicy: Never
      containers:
        - name: post-install-job
          image: "{{ .Values.database.image.repository }}:{{ .Values.database.image.tag}}"
          imagePullPolicy: {{ .Values.database.image.pullPolicy }}
          command: ['/bin/bash', '-c']
          args:
          - >
            mysql --host=$(MYSQL_DATABASE_HOST) --user=$(MYSQL_DATABASE_USER) --password=$(MYSQL_DATABASE_PASSWORD) &&
            mysql --host=$(MYSQL_DATABASE_HOST) --user=$(MYSQL_DATABASE_USER) --password=$(MYSQL_DATABASE_PASSWORD) --database=$(MYSQL_DATABASE) -e "CREATE TABLE test (id int AUTO_INCREMENT PRIMARY KEY, name CHAR(64));"  &&
            mysql --host=$(MYSQL_DATABASE_HOST) --user=$(MYSQL_DATABASE_USER) --password=$(MYSQL_DATABASE_PASSWORD) --database=$(MYSQL_DATABASE) -e "INSERT INTO test (name) VALUES ('helm'), ('kubernetes'), ('openshift'), ('docker');"
          env:
          - name: MYSQL_DATABASE_USER
            value: {{ .Values.mariadb.auth.username }}
          - name: MYSQL_DATABASE_PASSWORD
            value: {{ .Values.mariadb.auth.password }}
          - name: MYSQL_DATABASE_ROOT_PASSWORD
            value: {{ .Values.mariadb.auth.rootPassword }}
          - name: MYSQL_DATABASE
            value: {{ .Values.mariadb.auth.database }}
          - name: MYSQL_DATABASE_HOST
            value: {{ .Release.Name }}-mariadb

```

{{% /details %}}

Deploy the new hooks with the following command:

```bash
helm upgrade myapp ./mychart --namespace <namespace>
```


## Task {{% param sectionnumber %}}.4: Verify your hook

When you created your hooks, install or upgrade your helm release with `helm install ...` or `helm upgrade ...`. To verify if the data was created and persisted in the database use:

```s

{{% param cliToolName %}} --namespace <namespace> exec -it myapp-mariadb-0 -- mysql --host=localhost --user=acend --password=mysuperpassword123 --database=acenddb -e "SELECT * FROM test"  

+----+------------+
| id | name       |
+----+------------+
|  1 | helm       |
|  2 | kubernetes |
|  3 | openshift  |
|  4 | docker     |
+----+------------+

```


## Task {{% param sectionnumber %}}.5: Cleanup


Uninstall the app

```bash
helm uninstall myapp --namespace <namespace>
```
