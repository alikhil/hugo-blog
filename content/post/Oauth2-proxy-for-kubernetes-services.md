---
title: "Oauth2 Proxy for Kubernetes Services"
date: 2018-05-17T20:24:36+03:00
draft: true
coverImage: https://cdn-images-1.medium.com/max/2000/1*_MUGy2oo79pWpaBMFVtNsA.jpeg
coverMeta: out
coverSize: partial

thumbnailImage: "images/posts/k8s_logo.png"
thumbnailImagePosition: left

keywords:
- proxy
- authentication
- kubernetes
- google cloud platform

categories:
- devops

tags:
- oauth2-proxy
- authentication
- kubernetes
- google cloud
---

Hello, folks!

In this post, I will go through configuring [Bitly OAuth2 proxy](https://github.com/bitly/oauth2_proxy) in a kubernetes cluster.

<!--more-->

A few days ago I was configuring [SSO](https://en.wikipedia.org/wiki/Single_sign-on) for our internal dev-services in [KE Technologies](https://github.com/KazanExpress).

And I spent the whole to make it work properly, and at the end I decided that I will share my experience by writing this post, hoping that it will help others(and possibly me in the future) to go through this process.
<!-- toc -->

# What do we want?

We have internal services in our k8s cluster that we want to be accessible for developers.
It can be *kubernetes-dashboard* or *kibana* or anything else.

Before that we used Basic Auth, it's [easy to setup](https://banzaicloud.com/blog/ingress-auth/) in ingresses. But this approach has several disadvantages:

1. We need to share a single pair of *login* and *password* for all services among all developers
2. Developers will be asked to enter credentials each time when they access service first time

What we want is that developer will log in once and will have access to all other services without additional authentication.

So, a possible scenario could be:

1. Developers open https://kibana.example.com which is internal service
2. Browser redirects them to https://auth.example.com where they sign in
3. After successful authentication browser redirects them to https://kibana.example.com

# Preparation


## Kubernetes

First of all, we need a Kubernetes cluster. I will use the newly created cluster in **Google Cloud Platform** with version **1.8.10-gke.0**. If you have a cluster with configured ingress and https you can skip this step.

Then we need to install [**nginx ingress**](https://github.com/kubernetes/ingress-nginx) and [**kube lego**](https://github.com/jetstack/kube-lego). Let's do it using helm:

### Init helm

With RBAC:

```bash
# giving default accout admin role:
ACCOUNT=$(gcloud info --format='value(config.account)')

kubectl create clusterrolebinding owner-cluster-admin-binding \
    --clusterrole cluster-admin \
    --user $ACCOUNT


kubectl -n kube-system create sa tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```

without RBAC:
```bash
helm init
```

### Install nginx-ingress

```bash

helm install stable/nginx-ingress --name nginx-ing --namespace nginx-ing \
    --set controller.image.repository=gcr.io/google_containers/nginx-ingress-controller \
    --set controller.image.tag="0.9.0-beta.15"
    --set rbac.create=true # if RBAC is enabled in the cluster
    # see all options here: https://github.com/kubernetes/charts/blob/master/stable/nginx-ingress/values.yaml
```

After it's installed we can retrieve controller IP address:

```bash
kubectl --namespace nginx-ing get services -o wide -w nginx-ing-nginx-ingress-controller
```

and create DNS record to point our domain and subdomains to this IP address.

```txt
A       example.com     xxx.xxx.xx.xxx
CNAME   *.example.com   example.com
```

### Install kube-lego

```bash
helm install --name kube-lego stable/kube-lego --namespace kube-lego \
    --set config.LEGO_SUPPORTED_INGRESS_CLASS=nginx \
    --set config.LEGO_SUPPORTED_INGRESS_PROVIDER=nginx \
    --set config.LEGO_DEFAULT_INGRESS_CLASS=nginx \
    --set config.LEGO_URL=https://acme-v01.api.letsencrypt.org/directory \
    --set rbac.create=true \
    --set image.tag=0.1.5 \
    --set config.LEGO_LOG_LEVEL=debug
```

### Test it!

Let's run simple HTTP server as service and expose it using nginx ingress:

```bash
kubectl run simple-http --image=strm/helloworld-http --port=80

kubectl expose deployment simple-http --name example-service --port=80 --target-port=80 --type=NodePort
```


{{< codeblock "example-ing.yaml" "yml" >}}


apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: 'true'
spec:
  rules:
    - host: service.example.com
      http:
        paths:
          - backend:
              serviceName: example-service
              servicePort: 80
            path: /
  tls:
    - hosts:
        - "service.example.com"
      secretName: ing-tls
{{< /codeblock >}}

```bash
kubectl apply -f example-ing.yaml
```

Wait for a few seconds and open https://service.example.com and you should see something similar to this:

{{< image src="https://user-images.githubusercontent.com/7482065/40230488-575867a4-5aa0-11e8-907c-8878b0b951e6.png" title="Example">}}

## GitHub application

In this post, we will use GitHub accounts for authentication.

So, go to https://github.com/settings/applications/new and create new OAuth application

Fill **Authorization callback URL** field with https://auth.example.com/oauth2/callback where *example.com* is your domain name.

{{< image classes="[classes]" src="https://user-images.githubusercontent.com/7482065/40223994-7f7c7f94-5a8d-11e8-97db-9ca6e0809e0a.png" title="GitHub Application" >}}

After creating an application you will have **Client ID** and **Client Secret** which we will need in next step.

# Deploy OAuth Proxy

There are a lot of docker images for OAuth proxy, but we can not use them because they do not support domain white-listing. The problem is that such functionality has not implemented yet.

Actualy there are several PRs that solve that problem but seems to be they frozen for an unknown amount of time.

So, the only thing I could do is to merge one of the PRs to current master and build own image.

You also can use my image, but if you worry about security just clone [my fork](https://github.com/alikhil/oauth2_proxy) and build image yourself.


Let's create a namespace and set it as current:

```bash
kubectl create ns oauth-proxy

kns oauth-proxy # I am using kubectx tool -> https://github.com/ahmetb/kubectx

```

## Deploy secret

{{< codeblock "secret.yml" "yml" >}}
apiVersion: v1
kind: Secret
metadata:
  name: oauth-proxy-secret
  namespace: oauth-proxy
data:
  github-client-id: base64(YOUR_CLIENT_ID)
  github-client-secret: base64(YOUR_CLIENT_SECRET)
  cookie-secret: base64(random_string)

{{< /codeblock >}}

```bash
kubectl create -f secret.yml
```

## Deploy deployment

{{< codeblock "oauth-proxy.deployment.yml" "yml" >}}
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    k8s-app: oauth2-proxy
  name: oauth2-proxy
  namespace: oauth-proxy
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: oauth2-proxy
  template:
    metadata:
      labels:
        k8s-app: oauth2-proxy
    spec:
      containers:
      - name: oauth2-proxy
        image: alikhil/oauth2_proxy:2.2.2 
        imagePullPolicy: Always
        args:
        - --provider=github
        - --email-domain=*
        - --upstream=file:///dev/null
        - --http-address=0.0.0.0:4180
        - --whitelist-domain=.example.com
        - --cookie-domain=.example.com 
        # - --cookie-expire duration: expire timeframe for cookie (default 168h0m0s)
        # - --cookie-name string: the name of the cookie that the oauth_proxy creates (default "_oauth2_proxy")
        # - --cookie-refresh duration: refresh the cookie after this duration; 0 to disable
        # - --cookie-secret string: the seed string for secure cookies (optionally base64 encoded)
        env:
        - name: OAUTH2_PROXY_CLIENT_ID
          valueFrom:
            secretKeyRef:
              name: oauth-proxy-secret
              key: github-client-id
        - name: OAUTH2_PROXY_CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              name: oauth-proxy-secret
              key: github-client-secret
        - name: OAUTH2_PROXY_COOKIE_SECRET
          valueFrom:
            secretKeyRef:
              name: oauth-proxy-secret
              key: cookie-secret
        ports:
        - containerPort: 4180
          protocol: TCP

{{< /codeblock >}}


```bash
kubectl create -f oauth-proxy.deployment.yml
```

## Deploy service

{{< codeblock "oauth-service.yml" "yml" >}}

apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: oauth2-proxy
  name: oauth2-proxy
  namespace: oauth-proxy
spec:
  type: NodePort
  ports:
  - name: http
    port: 4180
    protocol: TCP
    targetPort: 4180
  selector:
    k8s-app: oauth2-proxy

{{< /codeblock >}}

```bash
kubectl create -f oauth-service.yml
```

## Deploy ingress

{{< codeblock "oauth-ing.yml" "yml" >}}

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: oauth2-proxy

  annotations:
    kubernetes.io/tls-acme: "true"
    kubernetes.io/ingress.class: "nginx"
    
spec:
  rules:
  - host: auth.example.com
    http:
      paths:
      - backend:
          serviceName: oauth2-proxy
          servicePort: 4180
        path: /oauth2
  tls:
  - hosts:
    - auth.example.com
    secretName: oauth-proxy-tls

{{< /codeblock >}}

```bash
kubectl create -f oauth-ing.yml
```

## Test it!

You can update ingress that we used while configuring nginx-ingress or create a new one:

{{< codeblock "example-ing.yml" "yml" >}}

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example
  annotations:
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: 'true'
    ingress.kubernetes.io/auth-url: https://auth.example.com/oauth2/auth
    ingress.kubernetes.io/auth-signin: https://auth.example.com/oauth2/start?rd=https://$host$request_uri$is_args$args
spec:
  rules:
    - host: service.example.com
      http:
        paths:
          - backend:
              serviceName: example-service
              servicePort: 80
            path: /
  tls:
    - hosts:
        - service.example.com
      secretName: ing-tls

{{< /codeblock >}}

```bash
kubectl apply -f example-ing.yml
```

Then visit service.example.com and you will be redirected to GitHub authorization page:

{{< image src="https://user-images.githubusercontent.com/7482065/40262777-feb43e2a-5b12-11e8-92dd-d7d066d61c50.png" title="GitHub Authorization page">}}

And once you authenticate, you will have access to all your services under ingress that point to auth.example.com until cookie expires.

And that's it! Now you can put any of your internal services behind ingress with OAuth.

## Resources

Here is a list of resources that helped me to go through this proccess first time:


* https://eng.fromatob.com/post/2017/02/lets-encrypt-oauth-2-and-kubernetes-ingress/
* https://www.midnightfreddie.com/oauth2-proxy.html
* https://thenewstack.io/single-sign-on-for-kubernetes-dashboard-experience/
