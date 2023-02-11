---
title: "6. Go templating"
weight: 6
sectionnumber: 6
---

In this lab we are going to learn how to use Go templating in Helm templates.


## Task {{% param sectionnumber %}}.1: Create a new Helm chart

Let's create a new Helm chart with the name `gotemplatechart` and remove all default templates from the `templates` folder.


### Solution

```bash
helm create gotemplatechart
rm -rf gotemplatechart/templates/*
```

{{% onlyWhen openshift %}}
{{% alert title="Warning" color="secondary" %}}
Don't forget to adapt the image name to an unprivileged one (`nginxinc/nginx-unprivileged:latest`) so OpenShift can run it. See [lab 2](../02/) for details.
{{% /alert %}}
{{% /onlyWhen %}}


## Task {{% param sectionnumber %}}.2: Add a ConfigMap template

As the template directory is completely empty, it's time to create our first template. For the purpose of this lab we're going to use a simple ConfigMap template. If you don't exactly understand what a ConfigMap is, consider reading the {{% onlyWhenNot openshift %}}[Kubernetes documentation on how to configure a pod to use a ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/){{% /onlyWhenNot %}}{{% onlyWhen openshift %}}[OpenShift documentation on how to configure a pod to use a ConfigMap](https://docs.openshift.com/container-platform/latest/authentication/configmaps.html#authentication-configmaps-consuming-configmap-in-pods){{% /onlyWhen %}}.

Create the template called `gotemplatechart/templates/configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
```

We can now render the template with the following command:

```bash
helm template gotemplatechart -s templates/configmap.yaml
```

If everything went ok we can deploy a release of the chart in our namespace:

```bash
helm upgrade -i gotemplaterelease gotemplatechart --namespace <namespace>
```


## Task {{% param sectionnumber %}}.3: The first Go template directive

As our first Go template directives we are going to add so-called built-in objects: the chart name and version `{{ .Chart.Name }}-{{ .Chart.Version }}`.

The template directive is enclosed in double curly braces `{{` and `}}`.

Update the `gotemplatechart/templates/configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: {{ .Chart.Name }}-{{ .Chart.Version }}
```

Rendering the new template again with `helm template gotemplatechart ...` (see task 2) will therefore result in the following output:

```yaml
# Source: gotemplatechart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: gotemplatechart-0.1.0
```

The directives `{{ .Chart.Name }}` and `{{ .Chart.Version }}` inject the name and version of the actual chart. But where are those values coming from?

Within those directives data is accessible through a data structure. The leading `.` represents the root of the object structure and is the entrypoint to access data in templates.
Built-in objects such as a chart, release, file, template, values and more are therefore accessible in a similar fashion. Check the official [Helm documentation about built-in objects](https://helm.sh/docs/chart_template_guide/builtin_objects/) for further and more in-depth information.

The `.Chart` data structure obviously comes from the `Chart.yaml` file and represents the values of this file.


## Task {{% param sectionnumber %}}.4: Add data from values.yaml

As you could have guessed by now, the `values.yaml` file allows us to configure values and parameters used during the rendering of the templates to replace strings, parameters, functions or even to control whether a part of a template is rendered at all. Let's add a couple more directives to our ConfigMap.

Remove the default content of the `values.yaml` and replace it with the following:

```yaml
favoriteColor: blue
```

Now also add your favorite color to be rendered in the ConfigMap as data under the key `myFavoriteColor`. Edit the `gotemplatechart/templates/configmap.yaml` file accordingly.

Use `helm template gotemplatechart ...` again (as in task 2) to see what your rendered Kubernetes resources look like.

{{% alert title="Note" color="primary" %}}
The output should look like this:

```
---
# Source: gotemplatechart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
...
data:
  ...
  myFavoriteColor: blue
```

{{% /alert %}}


### Solution Task {{% param sectionnumber %}}.4

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: {{ .Chart.Name }}-{{ .Chart.Version }}
  myFavoriteColor: {{ .Values.favoriteColor }}
```


## Task {{% param sectionnumber %}}.5: Structured data

As mentioned in task 3, data within the built-in objects can be structured and values can be nested.

Update the ConfigMap template so that the following `values.yaml` will result in the same rendered resource as in task 4.

```yaml
favorite:
  color: blue
```


### Solution Task {{% param sectionnumber %}}.5

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: {{ .Chart.Name }}-{{ .Chart.Version }}
  myFavoriteColor: {{ .Values.favorite.color }}
```

{{% alert title="Note" color="primary" %}}
The official [Helm best practices](https://helm.sh/docs/chart_best_practices/values/) suggest using flat values over nested ones:
>In most cases, flat should be favored over nested. The reason for this is that it is simpler for template developers and users.
{{% /alert %}}


## Template functions and pipelines

As for now we have learned how to place values unmodified within templates. There are cases where we want to do something with the value before we place it in a resource. With functions and pipelines we can achieve exactly that.

When injecting strings like e.g. our favorite color in a template, we want to quote the strings:

```
{{ .Values.favorite.color }} --> blue
{{ quote .Values.favorite.color }} --> "blue"
```

The quote function therefore adds double quotes around the value. Functions follow the syntax `functionName arg1 arg2 ...`.

Helm has over 60 functions available. Some are defined in the [Go template language](https://godoc.org/text/template), others in the [Sprig template library](https://godoc.org/github.com/Masterminds/sprig).

Similar to Linux pipes known from shell commands, e.g. `ps -aux | grep ps`, you can use pipes in Go templates as well:

```
{{ .Values.favorite.color | upper | quote }} --> "BLUE"
{{ .Values.favorite.color | b64enc | quote }} --> "Ymx1ZQ==" # blue base64 encoded
{{ .Values.favorite.color | repeat 2 | quote }} --> "BLUEBLUE"
{{ .Values.favorite.band | default "Pink Floyd" | quote }} --> "Pink Floyd" # the our values.yaml doesn't consist of the favorite --> band value therefore the default value is rendered
{{ printf "%s%s" .Release.Name .Chart.Name | quote }} --> "release-namegotemplatechart"
```


## Task {{% param sectionnumber %}}.6: Add functions and pipelines

Make the required changes to your ConfigMap template so that it renders to the following output:

```yaml
# Source: gotemplatechart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: "gotemplatechart-0.1.0"
  myFavoriteColor: "blue"
```


### Solution Task {{% param sectionnumber %}}.6

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: {{ printf "%s-%s" .Chart.Name .Chart.Version | quote }}
  myFavoriteColor: {{ .Values.favorite.color | quote }}
```


## Conditionals

If then else control structures are very common in templating languages like Go Templating and basically look like this

```yaml
{{ if condition as pipeline }}
  # do this
{{ else if condition as pipeline }}
  # do that
{{ else}}
  # do else
{{ end }}
```

{{% alert title="Note: Conditional Operators" color="primary" %}}
With `and`, `or`, `not`, `eq` being functions, conditions therefore look like this `{{ if and .Values.favorite.band (eq .Values.favorite.band "The Rolling Stones") }}`
For this condition to be true, the value `.Values.favorite.band` must be set and is set to "The Rolling Stones"

{{% /alert %}}


## Task {{% param sectionnumber %}}.7: Add a condition

Let's add an example condition to our ConfigMap:

* When the favorite drink is water, coke, beer or wine, add a new line to the ConfigMap data part `glass: true`
* When its coffee or tea: `mug: true`

We start with adding the favorite drink to your `values.yaml`

```yaml
favorite:
  color: blue
  drink: [replace with your favorite drink]
```

Now edit the ConfigMap template accordingly.


### Solution Task {{% param sectionnumber %}}.7

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: {{ printf "%s-%s" .Chart.Name .Chart.Version | quote }}
  myFavoriteColor: {{ .Values.favorite.color | quote }}
  {{ if and .Values.favorite.drink (or (eq .Values.favorite.drink "water") (eq .Values.favorite.drink "coke") (eq .Values.favorite.drink "beer") (eq .Values.favorite.drink "wine")) }}glass: true{{ else if and .Values.favorite.drink (or (eq .Values.favorite.drink "coffee") (eq .Values.favorite.drink "tea")) }}mug: true{{ end }}
```

This condition is mostly unreadable due to the fact that we need to make sure the spaces for the next key and value set are correct.

Check out [Helm's documentation about controlling whitespace](https://v2.helm.sh/docs/chart_template_guide/#controlling-whitespace) for more details on how to control whitespaces and adapt your ConfigMap.

A more readable version could look like this:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: {{printf "%s-%s" .Chart.Name .Chart.Version | quote}}
  myFavoriteColor: {{ .Values.favorite.color | quote }}
  {{- with .Values.favorite }}
  {{- if and .drink (or (eq .drink "water") (eq .drink "coke") (eq .drink "beer") (eq .drink "wine")) }}
  glass: true
  {{- else if and .drink (or (eq .drink "coffee") (eq .drink "tea")) }}
  mug: true
  {{- end }}
  {{- end }}
```

The `with` Operator allows you to set the current scope (`.`) to a particular object, in our case the favorite object.


## Loops in Helm templates

The `range` Operator allows you to implement loops in Helm templates. E.g. to iterate over a list of cities:

```yaml
favorite:
  color: blue
  drink: water
cities:
  - Bern
  - Zurich
  - Basel
  - Geneva
  - ...
```

We can implement the template as follows:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: {{printf "%s-%s" .Chart.Name .Chart.Version | quote}}
  myFavoriteColor: {{ .Values.favorite.color | quote }}
  {{- with .Values.favorite }}
  {{- if and .drink (or (eq .drink "water") (eq .drink "coke") (eq .drink "beer") (eq .drink "wine")) }}
  glass: true
  {{- else if and .drink (or (eq .drink "coffee") (eq .drink "tea")) }}
  mug: true
  {{- end }}
  {{- end }}
  cities: |-
    {{- range .Values.cities }}
    - {{ . | quote }}
    {{- end }}
```

Check out [Helm's documentation about developing templates](https://v2.helm.sh/docs/chart_template_guide/#the-chart-template-developer-s-guide) for more details on templating with Helm.


## Task {{% param sectionnumber %}}.8: MariaDB integration (optional)

Change the Helm chart from lab 3 so that the mariadb integration can be configured with a conditional parameter, e.g. `persistence.enabled`.
Consider changing the following resources:

* Service
* Deployment
* PersistentVolumeClaim
* Configuration in the application
