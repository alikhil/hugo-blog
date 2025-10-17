---
title: "kubectl-find - UNIX-find-like plugin to find resources and perform action on them"
date: 2025-10-17T19:27:24+03:00
categories:
- opensource
- devops
tags:
- kubernetes
- find
- fd
- kubectl
---

Recently, I have developed a plugin for `kubectl` inspired by UNIX `find` utility to find and perform action on resources. And few days ago number of stars in the repo reached 50! I think it's a good moment to tell more about the project.

<!--more-->

## The problem

As engineer who works with kubernetes everyday I use kubectl a lot. Actually, more than 50% of my terminal history commands are related to kubernetes. Here is a top 10 commands:

```shell
1	2079  46.7611%    kubectl
2	425   9.55915%    git
3	324   7.28745%    helm
4	156   3.50877%    cd
5	146   3.28385%    ssh
6	130   2.92398%    kctx
7	114   2.5641%     kns
8	80    1.79937%    gcloud-auth
9	78    1.75439%    curl
10	66    1.48448%    docker
...
```

> Run [this command](https://superuser.com/a/250230) if you are curious what about yours the most popular commands in terminal history.

I use kubectl to check status of the pods, delete orphaned resources, trigger sync on `ExternalSecrets` and much more. When I realized half my terminal history was just kubectl commands, I thought — there must be a better way to find things in Kubernetes without chaining pipes with `grep` / `awk` / `xargs`. And I imagined how nice it would be to have a UNIX `find`-like tool — something that lets you search for exactly what you need in the cluster and then perform actions directly on the matching resources. I searched for a krew plugin like this but there was not any. For that reason, I decided to develop [one](https://github.com/alikhil/kubectl-find)!

## Development

I used [sample-cli-plugin](https://github.com/kubernetes/sample-cli-plugin) as a starting point. Its clean repository structure and straightforward design make it a great reference for working with the Kubernetes API. Additionally, it allows easy reuse of the extensive Kubernetes client libraries.

Almost everything in the Kubernetes ecosystem is written in Go, and this plugin is no exception — which is great, as it allows building binaries for a wide range of CPU architectures and operating systems.

## Features overview

### Filters

#### Find by resource name using regex

```shell
kubectl fd pods -r dev
```

#### Find pods with restarted containers

```shell
kubectl fd pods --restarted
```

#### Find pods with image matching regex

```shell
kubectl fd pods --image bitnami
```

#### Find pods by status

```shell
kubectl fd pods --status Pending
```

#### Find pods running on nodes matching regex

```shell
kubectl fd pods --node '*spot'
```

#### Find any resource by custom condition

Use `jq` filter to find any resource by any custom condition. `kubectl-find` uses [gojq](https://github.com/itchyny/gojq) implementation of `jq`.

```shell
# find Services with trafficDistribution set to PreferClose
kubectl fd svc -j '.spec.trafficDistribution == "PreferClose"'

# find pods with unset nodeSelector
kubectl fd pods -j '.spec.nodeSelector == null' -A

# find pods with undefined resources
kubectl fd pods -j 'any( .spec.containers[]; .resources == {} )' -A
```

### Actions

By default, `fd` will print found resources to Stdout. However, there flags that you can provide to perform action on found resources:

- `--delete` - to delete them
- `--patch` - to patch with provided JSON
- `--exec` - to run command on pods

## Getting started

Use [krew](https://krew.sigs.k8s.io/) to install the plugin:

```shell
krew install fd
```


## What’s next

I’m currently working on adding:

- JSON/YAML output format
- More filters
- Saved queries

If you’re tired of writing long `kubectl | grep | xargs` chains, give `kubectl fd` a try — it’s already saved me countless keystrokes.

Check out the repo ⭐ [github.com/alikhil/kubectl-find](https://github.com/alikhil/kubectl-find)
and share your ideas or issues — I’d love to hear how you use it!

[![Star History Chart](https://api.star-history.com/svg?repos=alikhil/kubectl-find&type=date&legend=bottom-right)](https://github.com/alikhil/kubectl-find)