---
title: "Kubernetes"
date: 2026-05-02T17:12:38Z
description: "Kubernetes guides and notes about graceful shutdown, OAuth2 Proxy, pod resizing, kubectl plugins, and application delivery."
url: /kubernetes/
schemaType: "CollectionPage"
hidden: true
clearReading: true
showMeta: false
comments: false
showTags: false
showPagination: false
showSocial: false
showDate: false
---

I use this page to group my Kubernetes posts, from production reliability notes to small tools that make cluster work easier.

## Reliability and operations

- [Why Graceful Shutdown Matters in Kubernetes](/posts/graceful-shutdown/) - practical rollout behavior, SIGTERM handling, and test results.
- [Kubernetes In-Place Pod Resize](/posts/in-place-pod-resize/) - notes on resizing pod resources without recreating pods.

## Access and delivery

- [OAuth2-proxy: protect services in kubernetes](/posts/oauth2-proxy-protect-services-in-k8s/) - protecting internal services with OAuth2 Proxy and Pocket ID.
- [Oauth2 Proxy for Kubernetes Services](/posts/oauth2-proxy-for-kubernetes-services/) - older OAuth2 Proxy guide kept as historical context.
- [Deploy SPA application to Kubernetes](/posts/deploy-spa-to-k8s/) - deploying a single page app with Nginx, Docker, Service, and Ingress.

## Tools

- [kubectl-find](/tools/kubectl-find/) - project page for my UNIX-find-like `kubectl` plugin.
- [kubectl-find - UNIX-find-like plugin to find resources and perform action on them](/posts/kubectl-find-plugin-50-stars-anniversary/) - launch note and feature overview.
