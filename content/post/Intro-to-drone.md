---
title: "Introduction to Drone.io"
date: 2020-05-24T15:12:41+03:00
draft: true
thumbnailImage: https://cdn.dribbble.com/users/65539/screenshots/5547006/drone-logo_2x.png
# thumbnailImage: "images/posts/drone-logo.png"
thumbnailImagePosition: left

keywords:
- ci/cd
- tutorial
- drone.io

categories:
- devops
---

In this post I have tried to structure the content of my talks about [drone.io](https://drone.io) CI/CD platform. I hope, this will be useful for those who are not familiar with it.

<!--more-->

If you know Russian language, you can watch one of my talks about drone.io [here](https://www.youtube.com/watch?v=MSsHRo9CYvk) or [there](https://www.youtube.com/watch?v=mKT-bLdRGvQ).


# Overview

Drone.io is worth a closer look because:

- It is an [opensource](https://github.com/drone/drone) project, written in GoLang. You can check the sources, you can contribute.

- It is a container native. Everything executed in an isolated Docker container. All native and custom plugins are docker images. And drone.io can be [easily](/#installation) run using docker.

- Pipelines are simply configurable using drone.yml. You don’t need to learn new DSL. drone.yml’s structure is intuitive and it’s similar to the format of other popular platforms (Circle CI, Travis CI, Gitlab). Moreover, if you don’t like YML, drone supports Jsonnet and Starlark configuration languages.

- Support of popular git providers (GitHub, BitBucket, GitLab, Gogs, Gitea, etc).

- Powerful plugin system. Using them you can customize the scenario of your pipeline or extend the system’s functionality.

- Kubernetes support. You can run your pipelines in your kubernetes cluster.

# License

Drone.io has enterprise and community (open source) licenses.

[Here](https://drone.io/enterprise/features/) you can find the list of features available in both editions. As you see, the most interesting features are available only in the enterprise edition.