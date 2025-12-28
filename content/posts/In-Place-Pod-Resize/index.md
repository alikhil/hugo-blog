---
title: "Kubernetes In-Place Pod Resize"
date: 2025-12-28T21:14:12+03:00
categories:
- opensource
- devops
- note
tags:
- kubernetes
---

About six years ago, while operating a large Java-based platform in Kubernetes, I noticed a recurring problem: our services required significantly higher CPU and memory during application startup. Heavy use of Spring Beans and AutoConfiguration forced us to set inflated resource requests and limits just to survive bootstrap, even though those resources were mostly unused afterwards.

This workaround never felt right. As an engineer, I wanted a solution that reflected the actual lifecycle of an application rather than its worst moment.

<!--more-->

I opened an [issue](https://github.com/kubernetes/kubernetes/issues/83111)  in the Kubernetes repository describing the problem and proposing an approach to adjust pod resources dynamically without restarts. The issue received little discussion but quietly accumulated interest over time (13 👍 emoji reaction). Every few months, an automation bot attempted to mark it as stale, and every time, I removed the label. This went on for nearly six years...

Until the release of Kubernetes 1.35 where In-Place Pod Resize feature was [marked as stable](https://kubernetes.io/blog/2025/12/19/kubernetes-v1-35-in-place-pod-resize-ga/).

## What In-Place Pod Resize Brings

In-Place Pod Resize allows Kubernetes to update CPU and memory requests and limits without restarting pods, whenever it is safe to do so. This significantly reduces unnecessary restarts caused by resource changes, leading to fewer disruptions and more reliable workloads.

For applications whose resource needs evolve over time, especially after startup, this feature provides a long-missing building block.

## Impact on VerticalPodAutoscaler

The new `resizePolicy` field is configured at the pod spec level. While it is technically possible to change pod resources manually, doing so does not scale. In practice, this feature should be driven by a workload controller.

At the moment, the only controller that supports in-place pod resize is the Vertical Pod Autoscaler (VPA).

There are two enhancement proposals enable this behavior:

- [AEP-4016: Support for in place updates in VPA](https://github.com/kubernetes/autoscaler/tree/455d29039bf6b1eb9f784f498f28769a8698bc21/vertical-pod-autoscaler/enhancements/4016-in-place-updates-support) which introduces `InPlaceOrRecreate` update mode

- [AEP-7862: CPU Startup Boost](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler/enhancements/7862-cpu-startup-boost) which is about temporarily boosting pod by giving more cpu during pod startup. This is conceptually similar to the approach proposed in my original issue.

Here is an example of Deployment and VPA using both AEP features:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example
spec:
  replicas: 1
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
        - name: app
          image: my-heavy-java-app:stable
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 1000m
              memory: 1024Mi
            limits:
              cpu: 2000m
              memory: 2048Mi

---

apiVersion: "autoscaling.k8s.io/v1"
kind: VerticalPodAutoscaler
metadata:
  name: example-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: example
  updatePolicy:
    updateMode: "InPlaceOrRecreate"
  resourcePolicy:
    containerPolicies:
      - containerName: "app"
        minAllowed:
          cpu: "250m"
          memory: "512Mi"
        maxAllowed:
          cpu: "3000m"
          memory: "8192Mi"
        # The CPU boosted resources can go beyond maxAllowed.
        startupBoost:
          cpu:
            type: "Factor"
            quantity: "2"

```

With such configuration pod will have doubled cpu requests and limits during startup. During the boost period no resizing will happen.

Once the pod reaches the `Ready` state, the VPA controller scales CPU down to the currently recommended value.

After that, VPA continues operating normally, with the key difference that resource updates are applied in place whenever possible.

## Limitations

Does this feature fully solve the problem described above? Only partially.

First, most application runtimes still impose fundamental constraints. Java and Python runtimes do not currently support resizing memory limits without a restart. This limitation exists outside of Kubernetes itself and is tracked in the OpenJDK project via [an open ticket](https://bugs.openjdk.org/browse/JDK-8359211).

{{< figure align=center src="images/runtimes-supporting-in-place-memory-resize.png" title="In-Place Pod Resize in Kubernetes: Dynamic Resource Management Without Restarts - Tim Allclair & Mofi Rahman, Google" >}}

Second, Kubernetes does not yet support decreasing memory limits, even with in-place Pod Resize enabled. This is a known limitation documented in the enhancement proposal for [memory limit decreases](https://github.com/kubernetes/enhancements/tree/758ea034908515a934af09d03a927b24186af04c/keps/sig-node/1287-in-place-update-pod-resources#memory-limit-decreases).

As a result, while in-place Pod Resize effectively addresses CPU-related startup spikes, memory resizing remains an open problem.

## Final thoughts

In place Pod Resize gives a foundation for cool new features like StartupBoost and makes use of VPA more reliable. While important gaps remain, such as [memory decrease support](https://github.com/kubernetes/kubernetes/issues/135670) and [scheduling race condition](https://github.com/kubernetes/kubernetes/issues/126891), this change represents a meaningful step forward.

For workloads with distinct startup and steady-state phases, Kubernetes is finally beginning to model reality more closely.