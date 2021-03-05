---
title: "Installation for macOS"
weight: 4
type: docs
sectionnumber: 1
---

## Installation for macOS

Install the `helm` CLI binary on your system:

1. [Download the latest release](https://github.com/helm/helm/releases)
1. Unpack it (e.g. `tar -zxvf <filename>`)
1. Find the `helm` binary in the unpacked directory and move it to its desired destination (e.g. `mv darwin-amd64/helm ~/bin/`)
    * The desired destination should be listed in your $PATH environment variable (`echo $PATH`)

{{% onlyWhen mobi %}}


## Proxy configuration

{{% alert title="Note" color="primary" %}}
If you have direct access to the internet from your location, the proxy configuration is not required.
{{% /alert %}}

Set your HTTP proxy environment variables so that a chart repository can be added to your Helm repos in a later lab:

```bash
export HTTP_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
export HTTPS_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
```

Replace `<username`> and `<password>` with your credentials. If you have special characters in your password, escape them with their corresponding hexadecimal values according to [this article](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters).
{{% /onlyWhen %}}


## Verification

Now, [verify your installation](../04/).
