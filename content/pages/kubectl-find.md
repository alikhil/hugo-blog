---
title: "kubectl-find"
date: 2026-05-02T17:12:38Z
description: "kubectl-find is a UNIX-find-like kubectl plugin for searching Kubernetes resources and acting on matching objects."
url: /tools/kubectl-find/
schemaType: "SoftwareApplication"
applicationCategory: "DeveloperApplication"
operatingSystem: "Linux, macOS, Windows"
codeRepository: "https://github.com/alikhil/kubectl-find"
hidden: true
build:
  list: never
  render: always
clearReading: true
showMeta: false
comments: false
showTags: false
showPagination: false
showSocial: false
showDate: false
---

`kubectl-find` is a `kubectl` plugin inspired by UNIX `find`. It searches Kubernetes resources by name, status, age, image, node, and custom conditions, then can act on the matches.

## Install

```shell
krew install fd
```

## Common searches

Find pods by name:

```shell
kubectl fd pods -r dev
```

Find pods with restarted containers:

```shell
kubectl fd pods --restarted
```

Find pods with images matching a pattern:

```shell
kubectl fd pods --image bitnami
```

Find any resource with a custom `jq` condition:

```shell
kubectl fd svc -j '.spec.trafficDistribution == "PreferClose"'
```

## Actions

By default, `kubectl fd` prints matching resources. It can also act on matches:

- `--delete` - delete matching resources
- `--patch` - patch matching resources with JSON
- `--exec` - run a command in matching pods

## Links

- [GitHub repository](https://github.com/alikhil/kubectl-find)
- [Feature overview post](/posts/kubectl-find-plugin-50-stars-anniversary/)
