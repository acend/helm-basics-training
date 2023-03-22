---
title: "Optional: 7. Create your own chart"
weight: 7
sectionnumber: 7
---

In this section were going to show you how to modify a Helm chart from an existing Kubernetes deployment. To provide you an easy entry point, we have already prepared a Helm chart skeleton. This chart contains all necessary template files. With this chart, we want to deploy two java microservices, one which produces random data when it’s REST interface is called. The other microservice consumes then the data and exposes it to its endpoint.

```
+----------+                    +----------+
| producer +<-------------------+ consumer +
+----------+                    +----------+
```

Imagine you're being a java development team embracing the DevOps culture and your job is to deploy the two services you designed to your Kubernetes cluster.


## Task {{% param sectionnumber %}}.1: Get the chart skeleton

Clone the repository we have prepared for you containing all the needed template files to get you started.

```bash
git clone https://github.com/acend/helm-basic-chart.git
```

After cloning the chart you have following structure:

```
.
├── Readme.md
└── helm-basic-chart
    ├── Chart.yaml
    ├── templates
    │   ├── _helpers.tpl
    │   ├── consumer-deployment.yaml
    │   ├── consumer-ingress.yaml
    │   ├── consumer-service.yaml
    │   ├── producer-deployment.yaml
    │   ├── producer-ingress.yaml
    │   ├── producer-service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml
```

The template files are always prefixed with the according application (e.g. `consumer-deployment.yaml` and `producer-deployment.yaml`) and suffixed with the resource they represent. This is a best approach we like to follow in order to keep our files ordered by the applications and for readability purposes.

The setup is quite simple here, for each application we have a deployment, a service and an ingress. If you inspect the files in the folder you will realize that we have a lot of hard coded properties we maybe would like to change in order to bring the applications to multiple environments.

