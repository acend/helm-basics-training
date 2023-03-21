---
title: "6.7 Optional: Publish your chart"
weight: 67
sectionnumber: 67
---

In this lab we will learn how to publish our Helm chart and make it accessible to the public by using GitHub pages.

{{% alert title="Note" color="primary" %}}
To work through this lab, you need a personal GitHub account. See [Hosting chart repositories](https://helm.sh/docs/topics/chart_repository#hosting-chart-repositories) for a lot of different strategies how to serve a Helm repository.
{{% /alert %}}

GitHub pages provides an easy way to expose static files over HTTP(S) to the public. Everyone with a GitHub account can use this feature. The files to expose must be located in the `docs/` subdirectory or on a separate branch which can be configured in the repository settings.


## Task {{% param sectionnumber %}}.1: Linting your chart

Linting the Helm chart and fix the errors before packaging and publishing is a good practice:

```bash
helm lint ./mychart
```

The linter recommends to add an icon to the chart. We can safely ignore this hint :)

```
==> Linting mychart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```


## Task {{% param sectionnumber %}}.2: Package your chart

Helm chart packages are compressed `tgz` archives. Use the `helm package` command to create such an archive:

```bash
mkdir docs && cd docs
helm package ../mychart
```

This will create an archive with the name `mychart-0.1.0.tgz` inside the `docs` directory. The chart version is defined in `Chart.yaml` and will be appended to the filename.


```
❯ tree
.
├── docs
│   └── mychart-0.1.0.tgz
└── mychart
    ├── Chart.yaml
    ├── charts
    ├── templates
    │   ├── deployment-mariadb.yaml
    │   ├── deployment.yaml
    │   ├── _helpers.tpl
    │   ├── hpa.yaml
    │   ├── ingress.yaml
    │   ├── NOTES.txt
    │   ├── secret.yaml
    │   ├── serviceaccount.yaml
    │   ├── service.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml
```


## Task {{% param sectionnumber %}}.3: Create a new Github Repository

We will use GitHub pages to publish our packages and make them accessible to the public.

Replace `<GITHUB_USERNAME>` with your personal GitHub account.

```bash
USERNAME=<GITHUB_USERNAME>
git init
echo "charts/"> .gitignore
git config --global user.email "$USERNAME@foo.ch"
git config --global user.name "$USERNAME"
git add .
git commit -m "First chart version"
git branch -M main
git remote add origin git@github.com:$USERNAME/mycharts.git
git push -u origin main
```

{{% alert title="Note" color="primary" %}}
If you want to access your Git repository over HTTPS use the following url pattern for the origin `https://github.com/$USERNAME/mycharts.git`
{{% /alert %}}


## Task {{% param sectionnumber %}}.4: Activate GitHub Pages

On the GitHub website navigate to `Settings -> Pages` of your newly created repository `mycharts`. Select the `main` branch and `/docs` repository and click _Save_.

You will see a message with the URL under which the Chart repository will be exposed:

`https://<GITHUB_USERNAME>.github.io/mycharts/`


## Task {{% param sectionnumber %}}.5: Create an index file

The Helm repository must provide an `index.yaml` file which list all contained packages and provide some metadata about them. Create an index file inside the `docs` directory. Replace `<GITHUB_USERNAME>` with your personal GitHub account.

```bash
USERNAME=<GITHUB_USERNAME>
helm repo index . --url https://$USERNAME.github.io/mycharts/
```

This creates an index file with the following content:

```yaml
apiVersion: v1
entries:
  mychart:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2021-09-02T10:05:54.135132319+02:00"
    description: A Helm chart for Kubernetes
    digest: 43745683f4e51e95524042f96de0072d798473a63a6e1f0d116c60ac7ef784f3
    maintainers:
    - email: john@doe.org
      name: John Doe
    name: mychart
    type: application
    urls:
    - https://johndoe.github.io/mycharts/mychart-0.1.0.tgz
    version: 0.1.0
generated: "2021-09-02T10:05:54.134536373+02:00"
```


## Task {{% param sectionnumber %}}.6 Verifying your repository

Register your GitHub pages as a new Helm repository:

```bash
USERNAME=<GITHUB_USERNAME>
helm repo add mycharts https://$USERNAME.github.io/mycharts/
```

Verify that your published chart is found by Helm:


```bash
helm repo update
helm search repo mycharts/mychart
```

You should the the following output:

```
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
mycharts/mychart        0.1.0           1.16.0          A Helm chart for Kubernetes
```


## Task {{% param sectionnumber %}}.7 Installing chart from repository

When your newly created repository is successfully registered you can install the published chart with the following command:

```bash
helm upgrade -i myapp-from-repo mycharts/mychart
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
