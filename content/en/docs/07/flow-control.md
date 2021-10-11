---
title: "7.1 Flow Control"
weight: 71
sectionnumber: 71
---

Let's dive a bit deeper into the world of flow control in Helm. There are few elements in Helm templates that let us alter and control the flow in our structures:

* `if/else` for creating conditional blocks
* `with` to specify a scope
* `range`, which provides a "for each"-style loop

In our examples you have already crossed paths with the `if/else` and `with` construct. To be fair all these elements are fairly simple, yet very powerful.


### If/Else

The basic `if/else` pattern in Helm template looks like this:

```yaml

{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}

```


### With

The `with` keyword changes the scope of a defined block of code. The `.` always references the current scope at any time. With the keyword `with` you can easily adjust the scope of a defined block to simplify your template. Another often used behaviour is to test if variables are defined or not.

For example you want to define a `spec.backoffLimit` if it is defined in the `values.yaml`, otherwise obviously the field should be omitted. You can easily achieve this by using the `with` keyword:

```yaml

[...]
{{- with .Values.spec.backoffLimit }}
  backoffLimit: {{ . }}
{{- end }}

[...]

```


### Range

With the `range` keyword we can define for loop similar constructs:

```yaml

    {{- range .Values.someList }}
    - {{ . | title | quote }}
    {{- end }}    

```

You can see immediatly that the scope inside the range block changed. To access the parent scope, we can use `$`. For example `$.Values.someList` would reference the same list.

If you want to have information about the index in your loop, you can extend it like this:

```yaml

    {{- range $i, $a := .Values.someList }}
    - {{ . | title | quote }}
    {{- end }}    

```

Like this `$i` will hold the index during the iterations and `$a` will hold the context.


## Task {{% param sectionnumber %}}.1: Exercise

Will the flow control aspects Helm becomes a really powerful tool to template your resources. In the examples before we have seen how to setup and drop database tables. What if we want to be more dynamic and define the setup of the database in our values yaml and create the tables dynamically?

Define a data structure to describe the database and let Helm setup the database in a hook after releasing our database deployment.

If you have troubles starting the exercise take a look at the solution `values.yaml` file below and start from there.

{{% details title="Solution values.yaml" %}}

```yaml

tables:
  - name: first_table
    rows:
      - name: id
        type: int AUTO_INCREMENT PRIMARY KEY
      - name: value
        type: CHAR(64)
  - name: second_table
    rows:
      - name: id
        type: int AUTO_INCREMENT PRIMARY KEY
      - name: name
        type: CHAR(64)

```

{{% /details %}}

{{% details title="Solution templates/hook.yaml" %}}

```yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ .Release.Name }}-create-db"
  labels:
    app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
    app.kubernetes.io/instance: {{ .Release.Name | quote }}
    app.kubernetes.io/version: {{ .Chart.AppVersion }}
    helm.sh/chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
  annotations:
    "helm.sh/hook": post-install, post-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded, hook-failed
spec:
  backoffLimit: 3 
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
            {{- range .Values.tables }}
              mysql --host=$(MYSQL_DATABASE_HOST) --user=$(MYSQL_DATABASE_USER) --password=$(MYSQL_DATABASE_PASSWORD) --database=$(MYSQL_DATABASE) -e "CREATE TABLE IF NOT EXISTS {{ .name }} (
                {{- range $i, $a := .rows }}
                  {{ .name }} {{ .type }}{{ if ne $i (sub (len $.Values.tables ) 1)  }},{{ end }}
                {{- end }}
                );"
            {{- end }}
          env:
          - name: MYSQL_DATABASE_HOST
            value: {{ include "mychart.fullname" . }}-mariadb
          - name: MYSQL_DATABASE
            value: {{ .Values.database.databasename }}
          - name: MYSQL_DATABASE_PASSWORD
            valueFrom:
              secretKeyRef:
                key: database-password
                name: {{ include "mychart.fullname" . }}-mariadb
          - name: MYSQL_DATABASE_USER
            valueFrom:
              secretKeyRef:
                key: database-user
                name: {{ include "mychart.fullname" . }}-mariadb

```

{{% /details %}}
