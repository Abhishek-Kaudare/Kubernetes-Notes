- [**Ingress**](#ingress)
  - [**Terminology**](#terminology)
  - [**What is Ingress?**](#what-is-ingress)
  - [**Ingress Controller**](#ingress-controller)
  - [**Ingress Resource**](#ingress-resource)
- [**FAQ - What is the rewrite-target option?**](#faq---what-is-the-rewrite-target-option)
- [**References**](#references)

# **Ingress**
```bash
kubectl create ingress NAME --rule="host/path=service:port" [,tls[=secret]]
``` 

## **Terminology**

For clarity, this guide defines the following terms:

* `Node`: A worker machine in Kubernetes, part of a cluster.
* `Cluster`: A set of Nodes that run containerized applications managed by Kubernetes. For this example, and in most common Kubernetes deployments, nodes in the cluster are not part of the public internet.
* `Edge router`: A router that enforces the firewall policy for your cluster. This could be a gateway managed by a cloud provider or a physical piece of hardware.
* `Cluster network`: A set of links, logical or physical, that facilitate communication within a cluster according to the Kubernetes [networking model](https://kubernetes.io/docs/concepts/cluster-administration/networking/).
* `Service`: A Kubernetes Service that identifies a set of Pods using label selectors. Unless mentioned otherwise, Services are assumed to have virtual IPs only routable within the cluster network.

## **What is Ingress?**

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/#ingress-v1-networking-k8s-io) exposes HTTP and HTTPS routes from outside the cluster to [services](https://kubernetes.io/docs/concepts/services-networking/service/) within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.


Here is a simple example where an Ingress sends all its traffic to one Service:

![Ingress](https://d33wubrfki0l68.cloudfront.net/91ace4ec5dd0260386e71960638243cf902f8206/c3c52/docs/images/ingress.svg)

An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. An [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers) is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.

An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type [Service.Type=NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) or [Service.Type=LoadBalancer](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer).

## **Ingress Controller**
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller
          resources:
            limits:
              memory: "128Mi"
              cpu: "500m"
      args:
        - /nginx-ingress-controller
        - --configmap=$(POD_NAMESPACE)/nginx-configuration
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      ports:
        - name: http
          containerPort: 90
        - name: https
          containerPort: 443
```

## **Ingress Resource**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  creationTimestamp: "2022-06-05T14:54:21Z"
  generation: 1
  name: ingress-wear-watch
  namespace: app-space
  resourceVersion: "1351"
  uid: 09607977-686f-4cfe-942d-1414e982c740
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /watch
        pathType: Prefix
```
# **FAQ - What is the rewrite-target option?**

Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen [here](https://kubernetes.github.io/ingress-nginx/examples/). I would like to explain one such option that we will use in our labs. The [Rewrite](https://kubernetes.github.io/ingress-nginx/examples/rewrite/) target option.

Our `watch` app displays the video streaming webpage at `http://<watch-service>:<port>/`

Our `wear` app displays the apparel webpage at `http://<wear-service>:<port>/`

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/`

Without the `rewrite-target` option, this is what would happen:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/watch`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/wear`

Notice `watch` and `wear` at the end of the target URLs. The target applications are not configured with `/watch` or `/wear` paths. They are different applications built specifically for their purpose, so they don't expect `/watch` or `/wear` in the URLs. And as such the requests would fail and throw a `404` not found error.

To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the `rewrite-target` option. This rewrites the URL by replacing whatever is under `rules->http->paths->path` which happens to be `/pay` in this case with the value in `rewrite-target`. This works just like a search and replace function.

For example: `replace(path, rewrite-target)`In our case: `replace("/path","/")`

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

In another example given [here](https://kubernetes.github.io/ingress-nginx/examples/rewrite/), this could also be:

`replace("/something(/|$)(.*)", "/$2")`

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```
# **References**

1. [K8 Ingress Docs](https://kubernetes.io/docs/concepts/services-networking/ingress)
2. [K8 Ingress Docs - Path Types](https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types)

