---
title: "7. Extended Labs"
weight: 7
sectionnumber: 7
---

In the last couple of labs you've been learning a whole lot about helm and how helm works. It's now time to gain experience by using it in your day to day job.

The labs in this section are very open and give you a starting point on how to start to implement your own helm charts for your own applications.


## Task {{% param sectionnumber %}}.1: Explore the bitnami mongodb chart

The bitnami mongodb chart is one of the charts, that is very mature, nicely maintained and implements many solutions you're going to face while developing your own charts.

In this lab have a deeper look at the bitnami mongodb chart.

* Use the artifact hub as starting point: <https://artifacthub.io/packages/helm/bitnami/mongodb>
* You can find the source code for the chart [here]<https://github.com/bitnami/charts/tree/master/bitnami/mongodb>.

Explore the following parts of the chart:

* Explore the values.schema.json file. Those schemata define the possible values in the custom values.yaml
* The `architecture` value can be set either to `standalone` or `replicaset`. Explore the whole chart to understand how different the whole deployment will be, depending on which architecture was choosen.
  * deployment vs. statefulset
  * services
  * PVC
  * ...
* Try to find out how `extraEnvVars` can be defined in the values.yaml and then how those values are added to the actual deployment.
  * hint: for the `standalone` architekture check <https://github.com/bitnami/charts/blob/master/bitnami/mongodb/templates/standalone/dep-sts.yaml#L264>
  * hint2: the `common.tplvalues.render` template is in a chart dependency


## Task {{% param sectionnumber %}}.2: Create a new Helm chart for your own application

Start by creating a new helm chart from scratch `helm create <name>` and adapt the templates to your own use case.

