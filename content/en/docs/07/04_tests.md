---
title: "7.4 Test"
weight: 74
sectionnumber: 7.4
---

## Task {{% param sectionnumber %}}.1: Add a test to your chart

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

To install your release run:

```bash
helm upgrade -i myapp . --namespace <namespace>
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
