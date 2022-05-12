---
title: "4. Create your own chart"
weight: 4
sectionnumber: 4
---

In this section were going to show you how to create a Helm chart from an exisiting Kubernetes deployment

## Task {{% param sectionnumber %}}.1: Get the chart skeleton

```bash
git clone https://github.com/acend/helm-basic-chart.git
```

After cloning the chart you have following structure. The Helm template files are already there. 

```
.
├── Readme.md
└── helm-basic-chart
    ├── Chart.yaml
    ├── templates
    │   ├── _helpers.tpl
    │   ├── deployment-mariadb.yaml
    │   ├── deployment.yaml
    │   ├── ingress.yaml
    │   ├── secret-mariadb.yaml
    │   ├── serviceaccount.yaml
    │   ├── svc-mariadb.yaml
    │   ├── svc.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml
```


## Task {{% param sectionnumber %}}.2: Make the Chart more configurable

The `error-chart` chart contains some deliberate errors. Try to find all of these errors and fix them, then install the chart using `myrelease` as release name. You can use one or several ways shown above to do so.
The goal of this task is a successfully running `myrelease-error-chart` pod in your own namespace.


After cloning the Helm basic chart, we can start to modify the chart. Add following modifications:

* **deployment.yaml**
  * Make the `.spec.replicas` configurable 
  * Make the `.spec.containers.image` configurable. Make a seperate value for the image and the tag
* **deployment-mariadb.yaml**  
  * Make the `.spec.containers.image` configurable. Make a seperate value for the image and the tag
* **secret-mariadb.yaml**
  * Extract the `mariadb-password` and `mariadb-root-password` as value
* **ingress.yaml**
  * Extract `.spec.rules.host` as value
  * Extract `.spec.tls.hosts[0]` as value, use the same value as above
  * Also extract the `.spec.tls.hosts[0].secretName`


## Solution Task {{% param sectionnumber %}}.2


#### deployment.yaml
#### deployment-mariadb.yaml
#### secret-mariadb.yaml
#### ingress.yaml

### values.yaml

{{% onlyWhen openshift %}}


#### Deployment image

We've already encountered this one in [lab 2](../02/). Because of the used image, which wants to have root and privileged port numbers, it cannot be started on OpenShift.

We need to change the default image to an unprivileged one (`nginxinc/nginx-unprivileged:latest`) and also change the containerPort to `8080`.
{{% /onlyWhen %}}
