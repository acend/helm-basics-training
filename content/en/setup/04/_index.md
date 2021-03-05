---
title: "Verify the installation"
weight: 6
type: docs
sectionnumber: 1
---

## Verify the installation

To verify the installation, run the following command and check if `Version` is what you expected:

```bash
helm version
```

The output should be similar to this:

```
version.BuildInfo{Version:"v3.4.1", GitCommit:"c4e74854886b2efe3321e185578e6db9be0a6e29", GitTreeState:"clean", GoVersion:"go1.14.11"}
```


## First steps with helm

The `helm` binary has many subcommands. Invoke `helm --help` (or simply `-h`) to get a list of all subcommands; `helm <subcommand> --help` gives you detailed help about a subcommand.


## Next steps

When you're ready to go, head on over to the [labs](../../docs/) and begin with the training!
