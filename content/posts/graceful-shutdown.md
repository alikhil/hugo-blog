---
title: "Why Graceful Shutdown Matters in Kubernetes"
date: 2025-06-01T20:31:06+03:00
weight: 1
categories:
- sre
- devops

tags:
- go
- kubernetes
- tutorial
---



Have you ever deployed a new version of your app in Kubernetes and noticed errors briefly spiking during rollout? Many teams do not even realize this is happening, especially if they are not closely monitoring their error rates during deployments.

There is a common misconception in the Kubernetes world that bothers me. The official Kubernetes [documentation](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/) and most guides claim that "if you want zero downtime upgrades, just use rolling update mode on deployments". I have learned the hard way that this simply it is not true - rolling updates alone are **NOT enough** for true zero-downtime deployments.

And it is not just about deployments. Your pods can be terminated for many other reasons: scaling events, node maintenance, preemption, resource constraints, and more. Without proper graceful shutdown handling, any of these events can lead to dropped requests and frustrated users.

In this post, I will share what I have learned about implementing proper graceful shutdown in Kubernetes. I will show you exactly what happens behind the scenes, provide working code examples, and back everything with real test results that clearly demonstrate the difference.

<!--more-->

## The Problem: Hidden Errors During Pod Termination

{{< img align=center src="images/posts/k8s-graceful-shutdown.png" title="ChatGPT: draw funny picture of Kubernetes pod gracefully shutting down" >}}

If you are running services on Kubernetes, you have probably noticed that even with rolling updates (where Kubernetes gradually replaces pods), you might still see errors during deployment. This is especially annoying when you are trying to maintain "zero-downtime" systems.

When Kubernetes needs to terminate a pod (for any reason), it follows this sequence:

1. Sends a SIGTERM signal to your container
2. Waits for a grace period (30 seconds by default)
3. If the container does not exit after the grace period, it gets brutal and sends a SIGKILL signal

The problem? Most applications do not properly handle that SIGTERM signal. They just die immediately, dropping any in-flight requests. In the real world, while most API requests complete in 100-300ms, there are often those long-running operations that take 5-15 seconds or more. Think about processing uploads, generating reports, or running complex database queries. When these longer operations get cut off, that's when users really feel the pain.

### When Does Kubernetes Terminate Pods?

Rolling updates are just one scenario where your pods might be terminated. Here are other common situations that can lead to pod terminations:

* **Horizontal Pod Autoscaler Events**: When HPA scales down during low-traffic periods, some pods get terminated.

* **Resource Pressure**: If your nodes are under resource pressure, the Kubernetes scheduler might decide to evict certain pods.

* **Node Maintenance**: During cluster upgrades, node draining causes many pods to be evicted.

* **Spot/Preemptible Instances**: If you are using cost-saving node types like spot instances, these can be reclaimed with minimal notice.

All these scenarios follow the same termination process, so implementing proper graceful shutdown handling protects you from errors in all of these cases - not just during upgrades.

## Let's Test It: Basic vs. Graceful Service

Instead of just talking about theory, I built a small lab to demonstrate the difference between proper and improper shutdown handling. I created two nearly identical Go services:

* **Basic Service**: A standard HTTP server with no special shutdown handling
* **Graceful Service**: The same service but with proper SIGTERM handling

Both services:

* Process requests that take about 4 seconds to complete (intentionally configured for easier demonstration)
* Run in the same Kubernetes cluster with identical configurations
* Serve the same endpoints

I specifically chose a 4-second processing time to make the problem obvious. While this might seem long compared to typical 100-300ms API calls, it perfectly simulates those problematic long-running operations that occur in real-world applications. The only difference between the services is how they respond to termination signals.

To test them, I wrote a simple k6 script that hammers both services with requests while triggering rolling restart of service's deployment. Here is what happened:

### Basic Service: The Error-Prone One

```text
checks_total.......................: 695    11.450339/s
checks_succeeded...................: 97.98% 681 out of 695
checks_failed......................: 2.01%  14 out of 695

✗ status is 200
  ↳  97% — ✓ 681 / ✗ 14

http_req_failed....................: 2.01%  14 out of 696
```

### Graceful Service: The Reliable One

```text
checks_total.......................: 750     11.724824/s
checks_succeeded...................: 100.00% 750 out of 750
checks_failed......................: 0.00%   0 out of 750

✓ status is 200

http_req_failed........... ........: 0.00%  0 out of 751
```

The results speak for themselves. The basic service dropped 14 requests during the update (that is 2% of all traffic), while the graceful service handled everything perfectly without a single error.

You might think "2% it is not that bad" — but if you are doing several deployments per day and have thousands of users, that adds up to a lot of errors. Plus, in my experience, these errors tend to happen at the worst possible times.

## So How Do We Fix It? The Graceful Shutdown Recipe

After digging into this problem and testing different solutions, I have put together a simple recipe for proper graceful shutdown. While my examples are in Go, the fundamental principles apply to any language or framework you are using.

Here are the key ingredients:

### 1. Listen for SIGTERM Signals

First, your app needs to catch that SIGTERM signal instead of ignoring it:

```go
// Set up channel for shutdown signals
stop := make(chan os.Signal, 1)
signal.Notify(stop, os.Interrupt, syscall.SIGTERM)

// Block until we receive a shutdown signal
<-stop
log.Println("Shutdown signal received")
```

This part is easy - you are just telling your app to wake up when Kubernetes asks it to shut down.

### 2. Track Your In-Flight Requests

You need to know when it is safe to shut down, so keep track of ongoing requests:

```go
// Create a request counter
var inFlightRequests atomic.Int64

http.HandleFunc("/process", func(w http.ResponseWriter, r *http.Request) {
    // Increment counter when request starts
    inFlightRequests.Add(1)
    // do not forget to decrement when done!
    defer inFlightRequests.Add(-1)

    // Your normal request handling...
    time.Sleep(4 * time.Second)  // Simulating long-running work
})
```

This counter lets you check if there are still requests being processed before shutting down. it is especially important for those long-running operations that users have already waited several seconds for - the last thing they want is to see an error right before completion!

### 3. Separate Your Health Checks

Here is a commonly overlooked trick - you need different health check endpoints for liveness and readiness:

```go
// Track shutdown state
var isShuttingDown atomic.Bool

// Readiness probe - returns 503 when shutting down
http.HandleFunc("/ready", func(w http.ResponseWriter, r *http.Request) {
    if isShuttingDown.Load() {
        w.WriteHeader(http.StatusServiceUnavailable)
        fmt.Fprintf(w, "Shutting down, not ready")
        return
    }

    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, "Ready for traffic")
})

// Liveness probe - always returns 200 (we are still alive!)
http.HandleFunc("/alive", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, "I'm alive")
})
```

This separation is crucial. The readiness probe tells Kubernetes to stop sending new traffic, while the liveness probe says "do not kill me yet, I'm still working!"

### 4. The Shutdown Dance

Now for the most important part - the shutdown sequence:

```go
// Step 1: Mark service as shutting down
isShuttingDown.Store(true)

// Step 2: Let Kubernetes notice the readiness probe failing
time.Sleep(5 * time.Second)

// Step 3: Wait for in-flight requests to finish
for inFlightRequests.Load() > 0 {
    time.Sleep(1 * time.Second)
}

// Step 4: Finally, shut down the server gracefully
ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

if err := server.Shutdown(ctx); err != nil {
    log.Fatalf("Forced shutdown: %v", err)
}

```

I've found this sequence to be optimal. First, we mark ourselves as "not ready" but keep running. We pause to give Kubernetes time to notice and update its routing. Then we patiently wait until all in-flight requests finish before actually shutting down the server.

### 5. Configure Kubernetes Correctly

Do not forget to adjust your Kubernetes configuration:

```yaml
# Use different probes for liveness and readiness
livenessProbe:
  httpGet:
    path: /alive  # Always returns OK
    port: 8080
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready  # Returns 503 during shutdown
    port: 8080
  periodSeconds: 3
  failureThreshold: 2

# Give pods enough time to shut down gracefully
terminationGracePeriodSeconds: 30 # Stops routing traffic after 2 failed checks (6 seconds)
```

This tells Kubernetes to wait up to 30 seconds for your app to finish processing requests before forcefully terminating it.

## TL; DR; Quick Tips

If you are in a hurry, here are the key takeaways:

1. **Catch SIGTERM Signals**: Do not let your app be surprised when Kubernetes wants it to shut down.

2. **Track In-Flight Requests**: Know when it is safe to exit by counting active requests.

3. **Split Your Health Checks**: Use separate endpoints for liveness (am I running?) and readiness (can I take traffic?).

4. **Fail Readiness First**: As soon as shutdown begins, start returning "not ready" on your readiness endpoint.

5. **Wait for Requests**: Do not just shut down - wait for all active requests to complete first.

6. **Use Built-In Shutdown**: Most modern web frameworks have graceful shutdown options; use them!

7. **Configure Terminaton Grace Period**: Give your pods enough time to complete the shutdown sequence.

8. **Test Under Load**: You will not catch these issues in simple tests - you need realistic traffic patterns.

## Wrap Up: Is It Worth the Extra Code?

You might be wondering if adding all this extra code is really worth it. After all, we're only talking about a 2% error rate during pod termination events.

From my experience working with high-traffic services, I would say absolutely yes - for three reasons:

1. **User Experience**: Even small error rates look bad to users. Nobody wants to see "Something went wrong" messages, especially after waiting 10+ seconds for a long-running operation to complete.

2. **Cascading Failures**: Those errors can cascade through your system, especially if services depend on each other. Long-running requests often touch multiple critical systems.

3. **Deployment Confidence**: With proper graceful shutdown, you can deploy more frequently without worrying about causing problems.

The good news is that once you have implemented this pattern once, it is easy to reuse across your services. You can even create a small library or template for your organization.

In production environments where I have implemented these patterns, we have gone from seeing a spike of errors with every deployment to deploying multiple times per day with zero impact on users. that is a win in my book!

## Further Reading

If you want to dive deeper into this topic, I recommend checking out the article [Graceful shutdown and zero downtime deployments in Kubernetes](https://learnk8s.io/graceful-shutdown) from learnk8s.io. It provides additional technical details about graceful shutdown in Kubernetes, though it does not emphasize the critical role of readiness probes in properly implementing the pattern as we have discussed here.

---

For those interested in seeing the actual code I used in my testing lab, I've published it on [GitHub](https://github.com/alikhil/gracefull-shutdown-lab) with instructions for running the demo yourself.

Have you implemented graceful shutdown in your services? Did you encounter any other edge cases I didn't cover? Let me know in the comments how this pattern has worked for you!
