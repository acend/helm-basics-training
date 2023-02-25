---
title: "7.6 Deploy the Chart in another environment"
weight: 76
sectionnumber: 7.6
---

## Task {{% param sectionnumber %}}.1 prepare the value files

```yaml
replicaCount: 3

ingress:
  enabled: true
  hosts:
    - host: mychart-production-helm.ocp-internal.cloudscale.puzzle.ch
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []

database:
  enabled: true
  databaseuser: acend
  databasename: acenddb
  databasepassword: mysuperpassword123
  databaserootpassword: mysuperrootpassword123
  image:
    repository: mariadb
    pullPolicy: IfNotPresent
    tag: "10.5"
```

```yaml
replicaCount: 1

ingress:
  enabled: true
  hosts:
    - host: mychart-develop-helm.ocp-internal.cloudscale.puzzle.ch
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []

database:
  enabled: false
```


## Task {{% param sectionnumber %}}.8: Clean up

Uninstall the app

```bash
helm uninstall myapp-from-repo
```

The repository can be removed by using

```bash
helm repo remove mycharts
```
