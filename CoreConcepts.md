# **Table of Contents**

- [**Table of Contents**](#table-of-contents)
- [**Core Concepts**](#core-concepts)
  - [**Pods**](#pods)
  - [**Replica Sets**](#replica-sets)
  - [**Deployments**](#deployments)
  - [**Namespaces**](#namespaces)
    - [**When to Use Multiple Namespaces**](#when-to-use-multiple-namespaces)
    - [**Namespaces and DNS**](#namespaces-and-dns)
    - [**Working with Namespaces**](#working-with-namespaces)
      - [**Create Namespace**](#create-namespace)
      - [**List Namespaces**](#list-namespaces)
      - [**Setting the namespace for a request**](#setting-the-namespace-for-a-request)
      - [**Setting Request for all namespaces**](#setting-request-for-all-namespaces)
      - [**Setting the namespace preference**](#setting-the-namespace-preference)
    - [**Not All Objects are in a Namespace**](#not-all-objects-are-in-a-namespace)
  - [**Resource Quota**](#resource-quota)


# **Core Concepts**

## **Pods**

## **Replica Sets**

## **Deployments**

1. How many PODs exist on the system?
```bash
kubectl get pods
```
```bash
kubectl get po
```

2. How many ReplicaSets exist on the system?
```bash
kubectl get replicasets
```
```bash
kubectl get replicasets.apps
```
```bash
kubectl get rs
```

3. How many Deployments exist on the system?
```bash
kubectl get deployments
```
```bash
kubectl get deployments.apps
```

4. Get Image of any Pod in ReplicaSet/Deployment/Standalone
```bash
kubectl describe <deployments/replicasets/pods> <name> | grep -i image
```

5. Get Image Error of any Pods
```bash
kubectl describe pods <name> | grep -i image
```

6. Create a new Deployment using the `deployment-definition-1.yaml` file located at /root/
- Deployment File Example
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - name: busybox-container
        image: busybox888
        command:
        - sh
        - "-c"
        - echo Hello Kubernetes! && sleep 3600
```

- Create Deployment
```bash
kubectl apply -f deployment-definition-1.yaml
```
7. Create a new Deployment with the below attributes using your own deployment definition file.

    - Name: httpd-frontend
    - Replicas: 3
    - Image: httpd:2.4-alpine

- Create Deployment
```bash
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine
```

- Scale replicas to 3
```bash
kubectl scale deployment --replicas=3 httpd-frontend
```

- Validate
```bash
kubectl get deployments
```


## **Namespaces**

In Kubernetes, namespaces provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

Kubernetes starts with four initial namespaces:


- `default` : The default namespace for objects with no other namespace
- `kube-system` : The namespace for objects created by the **Kubernetes system**
- `kube-public` : This namespace is created automatically and is ***readable by all users*** (including those not authenticated). This namespace is mostly ***reserved for cluster usage***, in case that some resources should be visible and readable publicly throughout the whole cluster. **The public aspect of this namespace is only a convention, *not a requirement***.
- `kube-node-lease` : This namespace holds Lease objects associated with each node. **Node leases allow the kubelet to send heartbeats** so that the control plane can detect node failure.




### **When to Use Multiple Namespaces**


Namespaces are intended for use in **environments with many users spread across multiple teams, or projects**. For clusters with a few to tens of users, you should not need to create or think about namespaces at all. Start using namespaces when you need the features they provide.

- Namespaces provide a scope for names. 
- Names of resources need to be unique within a namespace, *but not across namespaces*. 
- Namespaces **cannot be nested** inside one another. 
- Each Kubernetes resource can only be in one namespace.
- Namespaces are a way to divide cluster resources between multiple users (via resource quota).

It is not necessary to use multiple namespaces to separate slightly different resources, such as different versions of the same software: **use labels to distinguish resources within the same namespace**.




### **Namespaces and DNS**
When you create a Service, it creates a corresponding DNS entry. This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means that if a container only uses <service-name>, it will resolve to the service which is local to a namespace. 

This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. 

If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).




### **Working with Namespaces**


#### **Create Namespace**
mamespace definition file:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <insert-namespace-name-here>
```
Command to create the namespace using definition file
```bash
kubectl create -f <namespace definition file path>
```
Command to create the namespace without using definition file
```bash
kubectl create namespace <insert-namespace-name-here>
```


#### **List Namespaces**
```bash
kubectl get namespace
```

```bash
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```



#### **Setting the namespace for a request**
When we run it request it runs for the **default namespace**. To set the namespace for a current request, use the `--namespace` flag.

```bash
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>

kubectl get pods --namespace=<insert-namespace-name-here>
```

Creating pods using following pod definition file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```
This command creates the pod in default namespace.
```bash
kubectl create -f <pod definition file name>
```
To create in particular namespace use the `--namespace` flag. 
```bash
kubectl create -f <pod definition file name>  --namespace=<insert-namespace-name-here>
```
To ensure that the resources are created in the same namespace add namespace to the pod definition file metadata

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: <insert-namespace-name-here>
  labels:
    app: myapp
    type: front-end
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```



#### **Setting Request for all namespaces**

To set the request for all namespaces, use the `--all-namespaces` flag.

```bash
kubectl get pods --all-namespaces
```


#### **Setting the namespace preference**
You can permanently save the namespace for all subsequent kubectl commands in that context.

```bash
kubectl config set-context --current --namespace=<insert-namespace-name-here>

# Validate it
kubectl config view --minify | grep namespace:
```

### **Not All Objects are in a Namespace** 
Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces. However namespace resources are not themselves in a namespace. And low-level resources, such as **nodes and persistentVolumes, are *not in any namespace***.

To see which Kubernetes resources are and aren't in a namespace:

```bash
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

## **Resource Quota**

Definition File:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

```bash
kubectl create -f <quota definition file path>
```
