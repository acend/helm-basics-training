---
title: "2. A simple chart"
weight: 2
---

In this lab we are going to create our very first Helm chart and deploy it.


## Task 1

First let's create our chart. Open your favorite terminal and make sure you're in the workspace for this lab, e.g. `cd ~/<workspace-helm-training>`:

```bash
helm create mychart
```

You now find a `mychart` directory with the newly created chart. It already is a valid and fully functional chart which deploys a nginx instance. Have a look at the generated files and their content. For an explanation of the files, visit the [Helm Developer Documentation](https://docs.helm.sh/developing_charts/#the-chart-file-structure). In a later section you'll find all the information about Helm templates.

{{% onlyWhen mobi %}}
Because you cannot pull the `nginx` container image on your cluster, you have to use the `docker-registry.mobicorp.ch/puzzle/k8s/kurs/nginx` container image. Change your `values.yaml` to match the following:

```yaml
[...]
image:
  repository: docker-registry.mobicorp.ch/puzzle/k8s/kurs/nginx
  tag: stable
  pullPolicy: IfNotPresent
[...]
```

{{% /onlyWhen %}}


## Task 2

Before actually deploying our generated chart, we can check the (to be) generated Kubernetes resources with the following command:

```bash
helm install --dry-run --debug --namespace <namespace> myfirstrelease ./mychart
```

Finally, the following command creates a new release and deploys the application:

```bash
helm install --namespace <namespace> myfirstrelease ./mychart
```

With `kubectl get pods --namespace <namespace>` you should see a new pod:

```bash
NAME                                     READY   STATUS    RESTARTS   AGE
myfirstrelease-mychart-6d4956b75-ng8x4   1/1     Running   0          2m21s
```

You can list the newly created Helm release with the following command:

```bash
helm ls --namespace <namespace>
```


## Task 3

Our freshly deployed nginx is not yet accessible from outside of the Kubernetes cluster. To expose it, we have to enable the ingress part in the `values.yaml`, which will then make helm create an ingress resource. Also make sure the application is accessible via TLS.


### Solution Task 3 Ingress

A look into the file `templates/ingress.yaml` reveals that the whole Ingress definition will only be part of the rendered resources if the condition `{{- if .Values.ingress.enabled -}}` is true:

```yaml
{{- if .Values.ingress.enabled -}}
{{- $fullName := include "mychart.fullname" . -}}
{{- $svcPort := .Values.service.port -}}
{{- if semverCompare ">=1.14-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: networking.k8s.io/v1beta1
{{- else -}}
apiVersion: extensions/v1beta1
{{- end }}
kind: Ingress
metadata:
  name: {{ $fullName }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ . }}
            backend:
              serviceName: {{ $fullName }}
              servicePort: {{ $svcPort }}
          {{- end }}
    {{- end }}
  {{- end }}
```

Thus we need to change this value inside our `values.yaml` file. This is also where we enable the TLS part:

```yaml
[...]
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: <namespace>.<appdomain>
      paths:
      - /
  tls:
    - secretName: <namespace>-<appdomain>
      hosts:
        - <namespace>.<appdomain>
[...]
```

{{% alert title="Note" color="primary" %}}
Make sure to set the proper value as hostname. `<appdomain>` will be provided by the trainer.
{{% /alert %}}

Apply the change by upgrading the release:

```bash
helm upgrade myfirstrelease ./mychart --namespace <namespace>
```

This will result in something similar to:

```
Release "myfirstrelease" has been upgraded. Happy Helming!
NAME: myfirstrelease
LAST DEPLOYED: Wed Dec  2 14:44:42 2020
NAMESPACE: <namespace>
STATUS: deployed
REVISION: 2
NOTES:
1. Get the application URL by running these commands:
  http://<namespace>.<appdomain>/
```

Check whether the ingress successfully was deployed, by accessing the URL `http://<namespace>.<appdomain>/`


## Task 4: NodePort

We can also expose the application using a `NodePort`.
First, set `ingress.enabled` back to `false` in your `values.yaml` file.
Now search for the Service type definition in your chart and make the change.


### Solution Task 4 NodePort

A look into the file `templates/service.yaml` reveals that the service type is set by the value `service.type`:

```yaml
[...]
spec:
  type: {{ .Values.service.type }}
[...]
```

Thus we need to change it inside our `values.yaml` file:

```yaml
[...]
service:
  type: NodePort
  port: 80
[...]
```

Apply the change by upgrading our release:

```bash
helm upgrade myfirstrelease ./mychart --namespace <namespace>
```

This will result in something similar to:

```
Release "myfirstrelease" has been upgraded. Happy Helming!
NAME: myfirstrelease
LAST DEPLOYED: Wed Dec  2 14:58:59 2020
NAMESPACE: <namespace>
STATUS: deployed
REVISION: 3
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace <namespace> -o jsonpath="{.spec.ports[0].nodePort}" services myfirstrelease-mychart)
  export NODE_IP=$(kubectl get nodes --namespace <namespace> -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

You will see in the following command's output when the service gets a `NodePort` (as we use `--watch` you have to terminate the command with CTRL-C):

```bash
kubectl get svc --namespace <namespace> --watch
```

nginx is now available at the given port number indicated by the `NodePort` and should display a welcome page when accessing it with `curl` or your browser of choice.


{{% alert title="Note" color="primary" %}}
Use either the output of the `helm upgrade` command, or `kubectl get node -o wide` to get a node ip address. Remember, [NodePort's](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) are open on any kubernetes node
{{% /alert %}}

{{% onlyWhen mobi %}}
In case you do not have permissions to list the nodes with `kubectl get node` please ask the trainer for a valid node ip address to access the welcome page.
{{% /onlyWhen %}}


## Task 5

An alternative way to set or overwrite values for charts we want to deploy is the `--set name=value` parameter. `--set name=value` can be used when installing a chart as well as upgrading.

Update the replica count of your nginx Deployment to 2 using `--set name=value`


### Solution Task 5

```bash
helm upgrade myfirstrelease --set replicaCount=2 ./mychart --namespace <namespace>
```

Values that have been set using `--set` can be reset by helm upgrade with `--reset-values`.


## Task 6

Have a look at the `values.yaml` file in your chart and study all the possible configuration params introduced in a freshly created chart.


## Task 7

To remove an application, simply remove the Helm release with the following command:

```bash
helm uninstall myfirstrelease --namespace <namespace>
```

Do this with our deployed release. With `kubectl get pods --namespace <namespace>` you should no longer see your application pod.
