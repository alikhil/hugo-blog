---
title: "Deploy SPA application to Kubernetes"
date: 2019-06-08T22:30:00+03:00
# draft: true

coverImage: https://user-images.githubusercontent.com/7482065/59222933-5f047a00-8bd3-11e9-87c1-e2571645832d.jpg
coverMeta: out
coverSize: partial
coverCaption: Photo by Calebe Miranda from Pexels

thumbnailImage: "images/posts/k8s_logo.png"
thumbnailImagePosition: left

keywords:
- spa
- tutorial
- k8s
- kubernetes
- docker

categories:
- devops
---

Hello, folks!

Today I want you to share with you tutorial on how to deploy your SPA application to Kubernetes. Tutorial is oriented for those don't very familiar with docker and k8s but want their single page application run in k8s.

### Dockerize the application

I expect that you have docker installed in your machine. If it isn't you can install it by following [official installation guide](https://docs.docker.com/install/).

As SPA project I will use [vue-realworld-example-app](https://github.com/gothinkster/vue-realworld-example-app) as SPA project. You can your own SPA project if you have one.

So, I have cloned it, installed dependencies and built:

```bash
git clone https://github.com/gothinkster/vue-realworld-example-app

yarn

yarn build
```

Next step is to decide how our application will be served. There are bunch of possible solutions but I decided to use [nginx](https://nginx.org/en/) since it recommends itself as one of the best http servers.

To serve SPA we need to return all requested files if they exist or otherwise fallback to index.html. To do so I wrote the following nginx config:

{{< codeblock "nginx.conf" "nginx" >}}
# ...
# other configs

server {
  listen 80;
  root /app;

  location / {
    alias /app/;
    try_files $uri /index.html;
  }

}
{{< /codeblock >}}


Full config file can be found in [my fork of the repo](https://github.com/alikhil/vue-realworld-example-app/blob/master/nginx.conf)

Then, we need to write **Dockerfile** for building image with our application. Here it is:

{{< codeblock "Dockerfile" "docker" >}}
FROM nginx
WORKDIR /root/
COPY ./dist /app
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
{{< /codeblock >}}

We assume that artifacts of build placed in the `dist` directory and so that during the docker build the content of `dist` directory copied into containers `/app` directory.

Now, we are ready to build it:

```bash
docker build -t alikhil/my-spa:0.1 .
```

And run it:

```bash
docker run -p 8080:80 alikhil/my-spa:0.1
```

Then if we open http://localhost:8080 we will see something similar to:

<img width="1279" alt="Screen Shot 2019-06-09 at 14 21 09" src="https://user-images.githubusercontent.com/7482065/59158361-e792f580-8ac1-11e9-9cd0-de309c697c12.png">

Cool! It works!

We will need to use our newly builded docker image to deploy to k8s. So, we need to make it available from the k8s cluster by pulling to some docker registry. I will push image to [DockerHub](http://hub.docker.com/):

```bash
docker push alikhil/my-spa:0.1
```

### Deploy to k8s

To run the application in k8s we will use `Deployment` resource type. Here it is:

{{< codeblock "deployment.yaml" "yml" >}}
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-spa
  labels:
    app: my-spa
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: my-spa
    spec:
      containers:
      - image: alikhil/my-spa:0.1
        name: spa
        ports:
          - containerPort: 80
        resources:
          limits:
            cpu: 150m
            memory: 250Mi
{{< /codeblock >}}

Then we create deployment by running `kubectl apply -f deployment.yaml` and newly created pods can found:

```bash
$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
my-spa-84b6dcd48d-mhv9f   1/1     Running   0          18s
```

Then we need to expose our app to the world. It can be done by using service of type NodePort or via Ingress. We will do it with Ingress. For that we will need service:

{{< codeblock "service.yaml" "yaml" >}}

apiVersion: v1
kind: Service
metadata:
  name: my-spa
  labels:
    app: my-spa
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
  selector:
    app: my-spa

{{< /codeblock >}}

And ingress itself:

{{< codeblock "ingress.yaml" "yaml" >}}
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-spa-ing
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  tls:
  - hosts:
    - my-spa.example.com
    secretName: my-spa-cert-secret
  rules:
  - host: my-spa.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: my-spa
          servicePort: 80

{{< /codeblock >}}

```bash
kubectl apply -f ingress.yaml -f service.yaml
```

And here it is! Our SPA runs in the k8s!

<img width="1253" alt="Screen Shot 2019-06-10 at 22 39 00" src="https://user-images.githubusercontent.com/7482065/59221731-a1788780-8bd0-11e9-806b-734048d3a291.png">
