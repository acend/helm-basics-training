---
title: "Installation for Windows"
weight: 3
type: docs
sectionnumber: 1
---

## Installation for Windows

Install the `helm` CLI binary on your system:

1. [Download the latest release](https://github.com/helm/helm/releases)
1. Unzip it
1. Find the `helm` binary in the unpacked directory and move it to its desired destination
    * The desired destination should be listed in your $PATH environment variable (`echo $PATH`)

{{% alert title="Note" color="primary" %}}
Windows quick hack: Copy the `helm` binary directly into the folder `C:\Windows`.
{{% /alert %}}

If the `$PATH` variable doesn't contain a suitable directory, it can be changed in the advanced system settings:

* [Windows 7](http://geekswithblogs.net/renso/archive/2009/10/21/how-to-set-the-windows-path-in-windows-7.aspx)
* [Windows 8](http://www.itechtics.com/customize-windows-environment-variables/)
* [Windows 10](http://techmixx.de/windows-10-umgebungsvariablen-bearbeiten/)

{{% onlyWhen mobi %}}


## Proxy configuration

{{% alert title="Note" color="primary" %}}
If you have direct access to the internet from your location, the proxy configuration is not required.
{{% /alert %}}

Set your HTTP proxy environment variables so that a chart repository can be added to your Helm repos in a later lab. It is recommended to set the lowercase and uppercase variables, as the helm command takes them all into account.

In Windows cmd:

```bash
setx HTTP_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
setx HTTPS_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
setx NO_PROXY="localhost,127.0.0.1,.mobicorp.ch,.mobicorp.test,.mobi.ch"
setx http_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
setx https_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
setx no_proxy="localhost,127.0.0.1,.mobicorp.ch,.mobicorp.test,.mobi.ch"
```

In Windows Powershell:

```bash
$env:HTTP_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
$env:HTTPS_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
$env:NO_PROXY="localhost,127.0.0.1,.mobicorp.ch,.mobicorp.test,.mobi.ch"
$env:http_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
$env:https_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
$env:no_proxy="localhost,127.0.0.1,.mobicorp.ch,.mobicorp.test,.mobi.ch"
```

In Git Bash:

```bash
export HTTP_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
export HTTPS_PROXY="http://<username>:<password>@dirproxy.mobi.ch:80"
export NO_PROXY="localhost,127.0.0.1,.mobicorp.ch,.mobicorp.test,.mobi.ch"
export http_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
export https_proxy="http://<username>:<password>@dirproxy.mobi.ch:80"
export no_proxy="localhost,127.0.0.1,.mobicorp.ch,.mobicorp.test,.mobi.ch"
```

Replace `<username`> and `<password>` with your credentials. If you have special characters in your password, escape them with their corresponding hexadecimal values according to [this article](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters).
{{% /onlyWhen %}}


## Verification

Now, [verify your installation](../04/).
