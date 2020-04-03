# HELM Techlab

In the guided hands-on techlab, we show the participants the Helm basics.

For more see [HELM Techlabs online](https://helm-techlab.k8s.puzzle.ch/).

## Content Sections

The Techlab content resides within the [content](content) directory.

The main part are the labs, which can be found at [content/labs](content/labs).

## Hugo

HELM Techlab is built using the static page generator [Hugo](https://gohugo.io/) and published under [helm-techlab.k8s.puzzle.ch](https://helm-techlab.k8s.puzzle.ch/).

The page uses the [dot theme](https://github.com/themefisher/dot) which is included as a Git Submodule.

After cloning the main repo, you need to initialize the submodule like this: 

```bash
git submodule update --init --recursive
``` 

## How to test locally
### Using Docker

Build the image:

```bash
docker build --build-arg HUGO_BASE_URL=http://localhost:8080/ -t dreng/helm-techlab:latest .
```

Run it locally:

```bash
docker run -i -p 8080:8080 dreng/helm-techlab
```


### Using Buildah and Podman

Build the image:

```bash
buildah build-using-dockerfile --build-arg HUGO_BASE_URL=http://localhost:8080/ -t dreng/helm-techlab:latest .
```

Run it locally:

```bash
podman run -i -p 8080:8080 localhost/dreng/helm-techlab
```


## Contributions

If you find errors, bugs or missing information please help us improve our techlab and have a look at the [Contribution Guide](CONTRIBUTING.md).
