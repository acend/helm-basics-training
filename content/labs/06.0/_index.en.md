---
title: "6.0 - Go Templating"
weight: 60
---

In this lab we are going to learn how to use go templating in the Helm templates. 

### Task 1: Create a new Helm Chart

Let's create a new Helm Chart with the name `gotemplatechart` and remove all default templates from the `templates` folder.

{{% collapse solution-1 "Solution Task 1" %}}

```bash
$ helm create gotemplatechart
$ rm -rf gotemplatechart/templates/*
```
{{% /collapse %}}

### Task 2: Create a new ConfigMap Template

As the template directory is completely empty, it's time to create our first template. For the purpose of this lab we're going to use a simple ConfigMap template. If you don't exactly understand what a ConfigMap is, consider reading the [Kubernetes Docs](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/).

Create the template called `gotemplatechart/templates/configmap.yaml`
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
$ helm template gotemplatechart -x templates/configmap.yaml
```

Or since it's a simple ConfigMap and how we learned in Lab 2 also deploy a release of it in our Namespace.

```bash
$ helm install gotemplatechart --name gotemplaterelease --namespace [USER]
```

{{% notice tip %}}
Make sure the Tiller Namespace Environment Variable (`export TILLER_NAMESPACE=[USER]`) is set to your Namespace or add the `--tiller-namespace [USER]` argument to the helm commands
{{% /notice %}}


### Task 3: Add the first Go Template Directive

As first Go Template Directives we're going to add so called Built-in Objects, the Chart Name and Version `{{ .Chart.Name }}-{{ .Chart.Version }}`

The template directive is enclosed in double curly brackets `{{` and `}}`

Update the `gotemplatechart/templates/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: {{ .Chart.Name }}-{{ .Chart.Version }}
```
Rendering the new template with `$ helm template gotemplatechart -x templates/configmap.yaml` will therefore result in the following output:

```yaml
---
# Source: gotemplatechart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: gotemplatelab-configmap
data:
  simplevalue: "Hello Helm"
  chartnameversion: gotemplatechart-0.1.0

```
The directive `{{ .Chart.Name }}` and `{{ .Chart.Version }}` injects the Name and Version of the actual Chart. But where are those values coming from?

Within those directives data is accessible through a data structure. The leading `.` represents the root of the object structure and is the Entrypoint to access data in templates.
Built-in Objects as Chart, Release, File, Template, Values and more are therefore accessible in similar fashion. Check the official [Helm Docs](https://v2.helm.sh/docs/chart_template_guide/#built-in-objects) for further and more in depth info.

The `.Chart` data structure obviously comes from the `Chart.yaml` file and represents the values of this file.

### Task 4: Add Data from the values.yaml

As you could have guessed by now, the `values.yaml` file allows us to configure values and parameter used during the rendering of the templates to replace Strings, Parameters, Functions or even to control whether a part of a template is rendered at all. Let's add a couple more directives to our ConfigMap.

Remove the default content of the `values.yaml` and replace it by the following:

```yaml
favoriteColor: blue
```

and then also add your Favorite Color to be rendered in the ConfigMap as data under the key `myFavoriteColor`, edit the `gotemplatechart/templates/configmap.yaml` File accordingly.

Use again `$ helm template gotemplatechart -x templates/configmap.yaml` to see what the output of your rendered K8S resource will look like.

{{% notice tip %}}
The Output should look like this
```yaml
---
# Source: gotemplatechart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
...
data:
  ...
  myFavoriteColor: blue
```
{{% /notice %}}

{{% collapse solution-4 "Solution Task 4" %}}

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
{{% /collapse %}}

### Task 5: Structured Data

As mentioned under Task 3, data within the Built-in Objects can be structured, values can be nested.

Update the ConfigMap Template, so that the following `values.yaml` will result in the same rendered Resource as in Task 4.

```yaml
favorite:
  color: blue
```

{{% collapse solution-5 "Solution Task 5" %}}

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
{{% /collapse %}}


{{% notice tip %}}
But ... the official [Helm Bestpractices](https://v2.helm.sh/docs/chart_best_practices/#values) suggest using flat values over nested: "In most cases, flat should be favored over nested. The reason for this is that it is simpler for template developers and users."
{{% /notice %}}

### Template Functions and Pipelines

As for now we've learned how to place values unmodified within templates. There are cases where we want to do something with the value before we place it in a resource. With functions and pipelines we can exactly achieve that.

When injecting Strings, like for example our favorite color in a template, we want to quote the Strings.

```
{{ .Values.favorite.color }} --> blue
{{ quote .Values.favorite.color }} --> "blue"
```
The quote function therefore adds double quotes around the value. Functions follow the syntax `functionName arg1 arg2 ...`

Helm has over 60 functions available. Some defined in the [Go Template Language](https://godoc.org/text/template) others in the [Sprig template library](https://godoc.org/github.com/Masterminds/sprig).

Similar to linux pipes known from shell commands eg. `ps -aux | grep ps` there are also pipelines available in Go Templates

```
{{ .Values.favorite.color | upper | quote }} --> "BLUE"
{{ .Values.favorite.color | b64enc | quote }} --> "Ymx1ZQ==" # blue base64 encoded
{{ .Values.favorite.color | repeat 2 | quote }} --> "BLUEBLUE"
{{ .Values.favorite.band | default "Pink Floyd" | quote }} --> "Pink Floyd" # the our values.yaml doesn't consist of the favorite --> band value therefore the default value is rendered
{{ printf "%s%s" .Release.Name .Chart.Name | quote }} --> "release-namegotemplatechart"
```

### Task 6: Update the ConfigMap Template with Functions and Pipelines

Make the required changes to your ConfigMap Template so that it renders to the following output:

```yaml
---
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
{{% collapse solution-6 "Solution Task 6" %}}

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
{{% /collapse %}}

### If - else if - else

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

{{% notice warning %}}

**Conditional Operators** Like `and`, `or`, `not`, `eq` are functions, conditions therefore look like this `{{ if and .Values.favorite.band (eq .Values.favorite.band "The Rolling Stones") }}`
For this condition to be true, the value `.Values.favorite.band` must be set and is set to "The Rolling Stones"

{{% /notice %}}

### Task 7: Add a Condition to the ConfigMap Template

Let's add an example condition to our ConfigMap:

* When the favorite drink is water, coke, beer or wine, add a new line to the configmap data part `glass: true`
* When its coffee or tea: `mug: true`

We start with adding the favorite drink to your `values.yaml`

```yaml
favorite:
  color: blue
  drink: [replace with your favorite drink]
```

Now edit the ConfigMap template accordingly

{{% collapse solution-7 "Solution Task 7" %}}

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
This Condition is mostly unreadable, due to the fact, that we need to make sure the spaces for the next key and value set are correct.

Check the [Helm Docs](https://v2.helm.sh/docs/chart_template_guide/#controlling-whitespace) for more details about how to control whitespaces and edit your Config Map.

A more readable Version could look like this
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

{{% /collapse %}}

### Loops in Helm Templates

The `range` Operator allows you to implement loops in Helm Templates. To iterate over a list of cities for example

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
we can implement the template as follows:

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

checkout the [Helm Developing Templates Docs](https://v2.helm.sh/docs/chart_template_guide/#the-chart-template-developer-s-guide) for more detailed infos about Templating with Helm.

### Task 8: Make mariadb integration for Lab 4 optional

Change the Helm Chart from Lab 4 so that the mariadb integration can be configured with a conditional parameter eg. `persistence.enabled`
* mariadb Service, Deployment
* PersistentVolumeClaim
* Configuration in the Spring Boot Application

