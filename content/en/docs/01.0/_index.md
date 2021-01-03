---
title: "1. Installation"
weight: 1
---

This guide shows you how to install the `helm` CLI tool. `helm` can be installed either from source or from pre-built binary releases.
We are going to use the pre-built releases.
`helm` binaries can be found on [Helm's release page](https://github.com/helm/helm/releases) for the usual variety of operating systems.


## Task 1

Install the `helm` CLI binary on your system:

1. [Download your desired version](https://github.com/helm/helm/releases)
1. Unpack it (e.g. `tar -zxvf helm-v3.4.1-linux-amd64.tar.gz`)
1. Find the `helm` binary in the unpacked directory and move it to its desired destination (e.g. `mv linux-amd64/helm /usr/local/bin/`)
    * The desired destination should be listed in your $PATH environment variable (`echo $PATH`)


To verify the installation, run the following command and check if `Version` is what you expected:

```bash
helm version
```

The output should be similar to this:

```
version.BuildInfo{Version:"v3.4.1", GitCommit:"c4e74854886b2efe3321e185578e6db9be0a6e29", GitTreeState:"clean", GoVersion:"go1.14.11"}
```

{{% onlyWhen mobi %}}


## Task 2

Set your HTTP proxy environment variables so a chart repository can be added to your Helm repos in a later lab:

```bash
# Linux
export HTTP_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
export HTTPS_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"

# Windows cmd
setx HTTP_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
setx HTTPS_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
setx http_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
setx https_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"

# Windows Powershell
$env:HTTP_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
$env:HTTPS_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
$env:http_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
$env:https_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
```

{{% alert title="Note" color="primary" %}}
If you have direct access to the internet from your location, the proxy configuration is not required.
{{% /alert %}}

Replace `<username`> and `<password>` with your credentials. If you have special characters in your password, escape them with their corresponding hexadecimal values according to [this article](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters).
{{% /onlyWhen %}}

You are now all set to start with the labs!
