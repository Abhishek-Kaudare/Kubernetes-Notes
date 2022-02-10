# **Certification Tip: Imperative Commands**

While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

- `-dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the `-dry-run=client` option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.
- `o yaml`: This will output the resource definition in YAML format on screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

## POD

**Create an NGINX Pod**

```shell
kubectl run nginx --image=nginx
```

**Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)**

```shell
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

## Deployment

**Create a deployment**

```shell
kubectl create deployment --image=nginx nginx
```

**Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)**

```shell
kubectl create deployment --image=nginx nginx --dry-run -o yaml
```

**Generate Deployment with 4 Replicas**

```shell
kubectl create deployment nginx --image=nginx --replicas=4
```

You can also scale a deployment using the `kubectl scale` command.

```shell
kubectl scale deployment nginx --replicas=4
```

**Another way to do this is to save the YAML definition to a file and modify**

```shell
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

You can then update the YAML file with the replicas or any other field before creating the deployment.

## Service

**Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**

```shell
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

(This will automatically use the pod's labels as selectors)

Or

```shell
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
```

(This will not use the pods labels as selectors, instead it will assume selectors as **app=redis.** [You cannot pass in selectors as an option.](https://github.com/kubernetes/kubernetes/issues/46191) So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

**Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:**

```shell
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
```

(This will automatically use the pod's labels as selectors, [but you cannot specify the node port](https://github.com/kubernetes/kubernetes/issues/25478). You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

```shell
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

**Reference:**

[https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)

## Formatting Output with kubectl

The default output format for all **kubectl** commands is the human-readable plain-text format.

The -o flag allows us to output the details in several different formats.

```shell
kubectl [command] [TYPE] [NAME] -o <output_format>
```

Here are some of the commonly used formats:

1. `-o json` Output a JSON formatted API object.
2. `-o name` Print only the resource name and nothing else.
3. `-o wide` Output in the plain-text format with any additional information.
4. `-o yaml` Output a YAML formatted API object.

Here are some useful examples:

**JSON format:**

```shell
kubectl create namespace test-123 --dry-run -o json
```
- **Output**
```json
{
    "kind": "Namespace",
    "apiVersion": "v1",
    "metadata": {
        "name": "test-123",
        "creationTimestamp": null
    },
    "spec": {},
    "status": {}
}
```

**YAML format:**

```shell
kubectl create namespace test-123 --dry-run -o yaml
```

- **Output**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: test-123
spec: {}
status: {}
```

**Output with wide (additional details):**

Probably the most common format used to print additional details about the object:
```shell
kubectl get pods -o wide
```
- **Output**
```shell
NAME      READY   STATUS    RESTARTS   AGE     IP          NODE     NOMINATED NODE   READINESS GATES
busybox   1/1     Running   0          3m39s   10.36.0.2   node01   <none>           <none>
ningx     1/1     Running   0          7m32s   10.44.0.1   node03   <none>           <none>
redis     1/1     Running   0          3m59s   10.36.0.1   node01   <none>           <none>
```

# Core Concepts

## Deployments Labs

1. How many PODs exist on the system?
```shell
kubectl get pods
```
```shell
kubectl get po
```

2. How many ReplicaSets exist on the system?
```shell
kubectl get replicasets
```
```shell
kubectl get replicasets.apps
```
```shell
kubectl get rs
```

3. How many Deployments exist on the system?
```shell
kubectl get deployments
```
```shell
kubectl get deployments.apps
```

4. Get Image of any Pod in ReplicaSet/Deployment/Standalone
```shell
kubectl describe <deployments/replicasets/pods> <name> | grep -i image
```

5. Get Image Error of any Pods
```shell
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
```shell
kubectl apply -f deployment-definition-1.yaml
```
7. Create a new Deployment with the below attributes using your own deployment definition file.

    - Name: httpd-frontend
    - Replicas: 3
    - Image: httpd:2.4-alpine

- Create Deployment
```shell
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine
```

- Scale replicas to 3
```shell
kubectl scale deployment --replicas=3 httpd-frontend
```

- Validate
```shell
kubectl get deployments
```
## Namespaces

In Kubernetes, namespaces provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

Kubernetes starts with four initial namespaces:

- `default` : The default namespace for objects with no other namespace
- `kube-system` : The namespace for objects created by the **Kubernetes system**
- `kube-public` : This namespace is created automatically and is ***readable by all users*** (including those not authenticated). This namespace is mostly ***reserved for cluster usage***, in case that some resources should be visible and readable publicly throughout the whole cluster. **The public aspect of this namespace is only a convention, *not a requirement***.
- `kube-node-lease` : This namespace holds Lease objects associated with each node. **Node leases allow the kubelet to send heartbeats** so that the control plane can detect node failure.

### When to Use Multiple Namespaces
Namespaces are intended for use in **environments with many users spread across multiple teams, or projects**. For clusters with a few to tens of users, you should not need to create or think about namespaces at all. Start using namespaces when you need the features they provide.

- Namespaces provide a scope for names. 
- Names of resources need to be unique within a namespace, *but not across namespaces*. 
- Namespaces **cannot be nested** inside one another. 
- Each Kubernetes resource can only be in one namespace.
- Namespaces are a way to divide cluster resources between multiple users (via resource quota).

It is not necessary to use multiple namespaces to separate slightly different resources, such as different versions of the same software: **use labels to distinguish resources within the same namespace**.

### Namespaces and DNS
When you create a Service, it creates a corresponding DNS entry. This entry is of the form `<service-name>.<namespace-name>.svc.cluster.local`, which means that if a container only uses <service-name>, it will resolve to the service which is local to a namespace. 

This is useful for using the same configuration across multiple namespaces such as Development, Staging and Production. 

If you want to reach across namespaces, you need to use the fully qualified domain name (FQDN).

### Working with Namespaces
#### **Create Namespace**
mamespace definition file:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <insert-namespace-name-here>
```
Command to create the namespace using definition file
```shell
kubectl create -f <namespace definition file path>
```
Command to create the namespace without using definition file
```shell
kubectl create namespace <insert-namespace-name-here>
```


#### **List Namespaces**
```shell
kubectl get namespace
```

```shell
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```



#### **Setting the namespace for a request**
When we run it request it runs for the **default namespace**. To set the namespace for a current request, use the `--namespace` flag.

```shell
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
```shell
kubectl create -f <pod definition file name>
```
To create in particular namespace use the `--namespace` flag. 
```shell
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

```shell
kubectl get pods --all-namespaces
```


#### **Setting the namespace preference**
You can permanently save the namespace for all subsequent kubectl commands in that context.

```shell
kubectl config set-context --current --namespace=<insert-namespace-name-here>

# Validate it
kubectl config view --minify | grep namespace:
```

### Not All Objects are in a Namespace 
Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces. However namespace resources are not themselves in a namespace. And low-level resources, such as **nodes and persistentVolumes, are *not in any namespace***.

To see which Kubernetes resources are and aren't in a namespace:

```shell
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

## Resource Quota

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

```shell
kubectl create -f <quota definition file path>
```