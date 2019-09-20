---
title: "Build own drone.io docker image"
date: 2019-09-20T12:30:00+03:00
# draft: true

coverImage: https://user-images.githubusercontent.com/7482065/65316587-eda96f80-dba2-11e9-81cf-2df2e0ce0309.png
coverMeta: out
coverSize: partial
coverCaption: Old drone logo

thumbnailImage: "images/posts/drone-logo.png"
thumbnailImagePosition: left

keywords:
- ci/cd
- tutorial
- drone.io
- docker

categories:
- devops
---

¡Hola, amigos!

In this post, I will quickly descibe how you can build your own [drone.io](https://drone.io) docker image.

<!--more-->

Drone is very popular container native CI/CD platform. Not long time ago, there was release of new [1.0 version](https://blog.drone.io/drone-1/) of drone. Which brang a lot of cool features and new [license](https://discourse.drone.io/t/licensing-and-subscription-faq/3839). The license tells that we can use Enterprise version of drone for free without any limits by building our own docker image if we are individuals or startup (read the licence for more detail).

So, how to build it?

# Instructions

First, clone the drone repo to your local machine.

```bash
git clone git@github.com:drone/drone.git
```

Second, checkout to version of drone you want to build. For example, I want to build [v1.3.1](https://github.com/drone/drone/tag/v1.3.1):

```bash
git checkout v1.3.1
```

We will use single dockerfile to build the image. To do so, we need to add extra step to existing dockerfile which is in `docker` directory. Let's say we want to build docker image for `linux` OS and `amd64` architecture, then we will edit `docker/Dockerfile.server.linux.amd64`.

<!-- Original drone docker images built in drone itself. To check how they built you can check `drone.yml`. -->

If you check the dockerfile you will see, that binaries are just copied into docker image during the build and they are built outside of the docker build. So, the step we will add to dockerfile is `go build` step.

To build the binary, we need to know what version of go is used for building binary in original docker image. We can find it in `drone.yml` **build** step. For version 1.3.1 of drone `golang:1.12.9` docker image is used for building binaries.

Then, we use same image to build binary in our dockerfile:



{{< codeblock "docker/Dockerfile.server.linux.amd64" "Dockerfile" >}}

FROM golang:1.12.9 as builder

WORKDIR /go/src/github.com/drone/drone
COPY . .

ENV GOOS linux
ENV GOARCH amd64
ENV CGO_ENABLED 1
ENV REPO github.com/drone/drone
ENV GO111MODULE on
RUN go build -tags nolimit -ldflags "-extldflags \"-static\"" -o release/linux/${GOARCH}/drone-server ${REPO}/cmd/drone-server

FROM alpine:3.9 as alpine
RUN apk add -U --no-cache ca-certificates

FROM alpine:3.9
EXPOSE 80 443
VOLUME /data

ENV GODEBUG netdns=go
ENV XDG_CACHE_HOME /data
ENV DRONE_DATABASE_DRIVER sqlite3
ENV DRONE_DATABASE_DATASOURCE /data/database.sqlite
ENV DRONE_RUNNER_OS=linux
ENV DRONE_RUNNER_ARCH=amd64
ENV DRONE_SERVER_PORT=:80
ENV DRONE_SERVER_HOST=localhost
ENV DRONE_DATADOG_ENABLED=true
ENV DRONE_DATADOG_ENDPOINT=https://stats.drone.ci/api/v1/series

COPY --from=alpine /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

COPY --from=builder /go/src/github.com/drone/drone/release/linux/amd64/drone-server /bin/
ENTRYPOINT ["/bin/drone-server"]

{{< /codeblock >}}

Then we build docker image like:

```bash
docker build -t alikhil/drone:1.3.1 -f docker/Dockerfile.server.linux.amd64 .
```

That's all! Now you can use own newly built docker image instead of official one if your use case meet license conditions.
