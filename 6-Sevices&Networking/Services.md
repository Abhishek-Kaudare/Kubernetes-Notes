- [**Service**](#service)
  - [**NodePort**](#nodeport)
  - [**ClusterIp**](#clusterip)
    - [**Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**](#create-a-service-named-redis-service-of-type-clusterip-to-expose-pod-redis-on-port-6379)
- [**References**](#references)

# **Service**
```bash
kubectl expose  (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type]

kubectl expose deployment nginx --port=443 --target-port=8443 --name=nginx-https --type=NodePort --name=serviceName
``` 
## **NodePort**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - port: 80 # Required
      targetPort: 80
      nodePort: 30008
  selector:
    app: myapp
    type: frontend
```

## **ClusterIp**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  ports:
    - port: 80 # Required
      targetPort: 80
  selector:
    app: myapp
    type: backend
```

### **Examples**

1. **Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**

    ```bash
    kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
    ```

    (This will automatically use the pod's labels as selectors)

    OR

    ```bash
    kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
    ```

    (This will not use the pods labels as selectors, instead it will assume selectors as **app=redis.** [You cannot pass in selectors as an option.](https://github.com/kubernetes/kubernetes/issues/46191) So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

--

2. **Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:**

    ```bash
    kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
    ```

    (This will automatically use the pod's labels as selectors, [but you cannot specify the node port](https://github.com/kubernetes/kubernetes/issues/25478). You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

    OR

    ```bash
    kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
    ```

    (This will not use the pods labels as selectors)

    Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

# **References**

1. [K8 Service Docs](https://kubernetes.io/docs/concepts/services-networking/service/)
2. [K8 Conventions](https://kubernetes.io/docs/reference/kubectl/conventions/)

