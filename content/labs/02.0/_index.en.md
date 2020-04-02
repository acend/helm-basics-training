---
title: "2.0 - Create a simple Chart"
weight: 20
---


### Task 1

In this lab, we use the `helm` cli to create our very first Helm Chart:

```bash
helm create mychart
```

This template is already a valid and fully functional chart which deploys NGINX. Have a look now on the generated files and their content. For an explanation of the files, visit the [Helm Developer Documentation](https://docs.helm.sh/developing_charts/#the-chart-file-structure). In a later Section you find all the information about Helm templates.


{{% notice tip %}}

**Die Mobiliar**

You can use the `docker-registry.mobicorp.ch/puzzle/k8s/kurs/nginx` container image instead of `nginx` which you cannot pull on your Kubernetes cluster.

So change your `values.yaml` file in the newly created `mychart`diretory.

```
[...]
image:
  repository: docker-registry.mobicorp.ch/puzzle/k8s/kurs/nginx
  pullPolicy: IfNotPresent
[...]
```

{{% /notice %}}



### Task 2

Before actually deploying our generated chart, we can check the (to be) generated Kubernetes ressources with the following command:

```bash
helm install --dry-run --debug --namespace [USER] mychart
```

Finally, the following command creates a new Release with the Helm chart and deploys the application::

```bash
helm install mychart --namespace [USER]
```

With kubectl get pods --namespace [USER] you should see a new Pod. You can list the newly created Helm release with 
the following command:

```bash
helm ls --namespace [USER]
```


### Task 3

Your deployed NGINX is not yet accessible from external. To expose it, you have to change the Service Type to NodePort. Search 
now for the service type definition in your chart and make the change. You can apply your change with the following command:

```bash
helm upgrade [RELEASE] --namespace [USER] mychart
```

As soon as the Service has a NodePort, you will see it with the following command (As we use -w (watch) you have to terminate the command with CTRL-C):

```bash
kubectl get svc --namespace [USER] -w
```

NGINX is now available at the given NodePort and should display a welcome-page when accessing it with curl or you can also open the page in your browser:


### Task 4

To remove an application, you can simply remove the Helm release with the following command:

```bash
helm delete [RELEASE]
```

With `kubectl get pods --namespace [USER]` you should now longer see your application Pod.