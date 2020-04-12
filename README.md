# Helm Techlab

In this guided hands-on techlab, we show participants the Helm's basics.

For more see [Helm Techlabs online](https://helm-techlab.k8s.puzzle.ch/).


## Content Sections

The Techlab content resides within the [content](content) directory.

The main part are the labs, which can be found at [content/labs](content/labs).


## Hugo

Helm Techlab is built using the static page generator [Hugo](https://gohugo.io/) and published at [helm-techlab.k8s.puzzle.ch](https://helm-techlab.k8s.puzzle.ch/).

The page uses the [dot theme](https://github.com/themefisher/dot) which is included as a Git Submodule.

After cloning the main repo, you need to initialize the submodule like this: 

```bash
git submodule update --init --recursive
``` 

## How to test locally
### Using Docker

Build the image:

```bash
docker build -t dreng/helm-techlab:latest .
```

Run it locally:

```bash
docker run --rm --interactive --publish 8080:8080 dreng/helm-techlab
```


### Using Buildah and Podman

Build the image:

```bash
buildah build-using-dockerfile -t dreng/helm-techlab:latest .
```

Run it locally with the following command. Beware that `--rmi` automatically removes the built image when the container stops, so you either have to rebuild it or remove the parameter from the command.

```bash
podman run --rm --rmi --interactive --publish 8080:8080 localhost/dreng/helm-techlab
```


## Contributions

If you find errors, bugs or missing information please help us improve our techlab and have a look at the [Contribution Guide](CONTRIBUTING.md).
