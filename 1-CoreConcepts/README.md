# **Table of Contents**

- [**Table of Contents**](#table-of-contents)
- [**Pods**](#pods)
  - [**1. Creating Pods**](#1-creating-pods)
    - [**Pod Definition File**](#pod-definition-file)
  - [**2. Editing Existing Pods**](#2-editing-existing-pods)
  - [**3. Create just a pod via imperative command with label**](#3-create-just-a-pod-via-imperative-command-with-label)
  - [**4. Create pod manifest for nginx**](#4-create-pod-manifest-for-nginx)
  - [**5. Get pod info**](#5-get-pod-info)
  - [**6. Get more details of pods**](#6-get-more-details-of-pods)
  - [**7. Update an existing pod from changed source yaml**](#7-update-an-existing-pod-from-changed-source-yaml)
  - [**8. Get running pod defintion from k8s and edit/update**](#8-get-running-pod-defintion-from-k8s-and-editupdate)
  - [**9. Edit running pod definition directly**](#9-edit-running-pod-definition-directly)
  - [**10. Delete all pods in default namespace**](#10-delete-all-pods-in-default-namespace)
  - [**11. List pods with selector**](#11-list-pods-with-selector)
  - [**12. Shell into (if possible) a running pod**](#12-shell-into-if-possible-a-running-pod)
  - [**13. Run '/bin/sh' command in a pod, and delete the pod after running the command**](#13-run-binsh-command-in-a-pod-and-delete-the-pod-after-running-the-command)
- [**Replica Sets**](#replica-sets)
  - [**1. Create a replicaset**](#1-create-a-replicaset)
    - [**Replica Set Definition File**](#replica-set-definition-file)
  - [**2. List all replicaset**](#2-list-all-replicaset)
  - [**3. Delete a replicaset**](#3-delete-a-replicaset)
  - [**4. Scale a replicaset to a 'N' number of pods**](#4-scale-a-replicaset-to-a-n-number-of-pods)
  - [**5. Edit running replicatset definition directly**](#5-edit-running-replicatset-definition-directly)
  - [**6. Update a replicaset with the replace command via definition file**](#6-update-a-replicaset-with-the-replace-command-via-definition-file)
  - [**7. Get definition file from kubectl**](#7-get-definition-file-from-kubectl)
- [**Deployments**](#deployments)
  - [**1. Check how many PODs exist on the system**](#1-check-how-many-pods-exist-on-the-system)
  - [**2. Check how many ReplicaSets exist on the system**](#2-check-how-many-replicasets-exist-on-the-system)
  - [**3. Check how many Deployments exist on the system**](#3-check-how-many-deployments-exist-on-the-system)
  - [**4. Get Image of any Pod in ReplicaSet/Deployment/Standalone**](#4-get-image-of-any-pod-in-replicasetdeploymentstandalone)
  - [**5. Get Image Error of any Pods**](#5-get-image-error-of-any-pods)
  - [**6. Create a new Deployment using the `deployment-definition-1.yaml` file located at /root/**](#6-create-a-new-deployment-using-the-deployment-definition-1yaml-file-located-at-root)
    - [**Deployment Definition File**](#deployment-definition-file)
  - [**7. Create a new Deployment with the below attributes using your own deployment definition file.**](#7-create-a-new-deployment-with-the-below-attributes-using-your-own-deployment-definition-file)
- [**Namespaces**](#namespaces)
  - [**Kuberenetes and Namespace**](#kuberenetes-and-namespace)
    - [**When to Use Multiple Namespaces**](#when-to-use-multiple-namespaces)
    - [**Namespaces and DNS**](#namespaces-and-dns)
  - [**1. Create a new namespace**](#1-create-a-new-namespace)
  - [**2. List Namespaces**](#2-list-namespaces)
  - [**3. Setting the namespace for a request**](#3-setting-the-namespace-for-a-request)
  - [**4. Setting Request for all namespaces**](#4-setting-request-for-all-namespaces)
  - [**5. Setting the namespace preference**](#5-setting-the-namespace-preference)
  - [**Not All Objects are in a Namespace**](#not-all-objects-are-in-a-namespace)
- [**Resource Quota**](#resource-quota)
  - [**Resource Quota Definition File**](#resource-quota-definition-file)
    - [**Compute Quota**](#compute-quota)
    - [**Count Quota**](#count-quota)
  - [**Imperative Commands**](#imperative-commands)
  - [**Compute Resource Quota**](#compute-resource-quota)
  - [**Storage Resource Quota**](#storage-resource-quota)
  - [**Object Count Quota**](#object-count-quota)


# **Pods**
## **1. Creating Pods**

### **Pod Definition File**

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: myapp-pod
  labels:
    app: myapp

spec:
  containers:
    - name: nginx-container
      image: nginx

    - name: backend-container
      image: redis
```

## **2. Editing Existing Pods**

In any of the practical quizzes if you are asked to **edit an existing POD**, please note the following:

- If you are given a pod definition file, edit that file and use it to create a new pod.

- **If you are not given a pod definition file**, you may extract the definition to a file using the below command:
  
  ```shell
  kubectl get pod <pod-name> -o yaml > pod-definition.yaml
  ```
  
  Then edit the file to make the necessary changes, delete and re-create the pod.

- Use the `kubectl edit pod <pod-name>` command to edit pod properties.

## **3. Create just a pod via imperative command with label**

```shell
$ kc run nginx --image=nginx -l tier=frontend
```

## **4. Create pod manifest for nginx**

```shell
$ kc run nginx --image=nginx --dry-run -o yaml
```

## **5. Get pod info**

```shell
$ kc describe pod nginx-pod
```

## **6. Get more details of pods**

```shell
$ kc get pod -o wide
```

## **7. Update an existing pod from changed source yaml**

```shell
$ kc apply -f redis-pod.yaml
```

## **8. Get running pod defintion from k8s and edit/update**

```shell
$ kc get po <pod-name> -o yaml > pod-definition.yaml
$ vim pod-definition.yaml
$ kc apply -f pod-definition.yaml
```

## **9. Edit running pod definition directly**

```shell
$ kc edit po <pod-name>
```

## **10. Delete all pods in default namespace**

```shell
$ kc delete po --all
```

## **11. List pods with selector**
```shell
$ kc get pod -l key1=value1,key2=value2
```

## **12. Shell into (if possible) a running pod**

```shell
$ kc exec -it <pod-name> -- /bin/sh (or /bin/bash, if available)
```

## **13. Run '/bin/sh' command in a pod, and delete the pod after running the command**
```shell
$ kc run <pod-name> --image=busybox  --restart=Never -it --rm command -- /bin/sh
# -it will help in seeing the output, --rm will immediately delete the pod after it exits
```

# **Replica Sets**
A template section under spec can define a child resource, e.g., a pod resource under replicaSet's `spec.template` section. Do not include apiVersion or kind in the template.

A replicaset requires the selector section with `matchLabels` defining the pod label to replicate on. 

It is still required to include the pod template even if the pods already exist before creating the replicaset, as any scale-up of more or failed pods will be done from this template.

A ReplicaSet is linked to its Pods via the Pods' `metadata.ownerReferences` field, which specifies what resource the current object is owned by. All Pods acquired by a ReplicaSet have their owning ReplicaSet's identifying information within their `ownerReferences` field. It's through this link that the ReplicaSet knows of the state of the Pods it is maintaining and plans accordingly.
## **1. Create a replicaset**

### **Replica Set Definition File**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 5
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: nginx
```
```shell
$ kc create -f replicaset-definition.yaml
```

## **2. List all replicaset**

```shell
$ kc get replicaset
```

## **3. Delete a replicaset** 
<div style="text-align: right"> <span style="color:blue"> *Deletes all underlying pods </span> </div>

```shell
$ kc delete replicaset <replicaset-name> 
```

## **4. Scale a replicaset to a 'N' number of pods**

```shell
$ kc scale --replicas=N rs <replicaset-name>
```

## **5. Edit running replicatset definition directly**

```shell
$ kc edit rs <replicaset>
```

## **6. Update a replicaset with the replace command via definition file**

```shell
$ kc replace -f replicaset-definition.yaml
```

## **7. Get definition file from kubectl**

```shell
$ kc get rs ngnix-replicaset -o yaml > nginx-replicaset.yaml
```
# **Deployments**

## **1. Check how many PODs exist on the system**
```bash
kubectl get pods
```
```bash
kubectl get po
```

## **2. Check how many ReplicaSets exist on the system**
```bash
kubectl get replicasets
```
```bash
kubectl get replicasets.apps
```
```bash
kubectl get rs
```

## **3. Check how many Deployments exist on the system**
```bash
kubectl get deployments
```
```bash
kubectl get deployments.apps
```
```bash
kubectl get deploy
```

## **4. Get Image of any Pod in ReplicaSet/Deployment/Standalone**
```bash
kubectl describe <deployments/replicasets/pods> <name> | grep -i image
```

## **5. Get Image Error of any Pods**
```bash
kubectl describe pods <name> | grep -i image
```

## **6. Create a new Deployment using the `deployment-definition-1.yaml` file located at /root/**

### **Deployment Definition File**
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
Create Deployment
```bash
kubectl apply -f /root/deployment-definition-1.yaml
```
## **7. Create a new Deployment with the below attributes using your own deployment definition file.**

    - Name: httpd-frontend
    - Replicas: 3
    - Image: httpd:2.4-alpine
    - Expose port 5771
    - Add Command sh -c 'echo Hello Kubernetes! && sleep 3600'
    - Scale replicas to 4

- Create Deployment
```bash
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3 --port=5771 -- sh -c 'echo Hello Kubernetes! && sleep 3600'
```

- Scale replicas to 4
```bash
kubectl scale deployment --replicas=4 httpd-frontend
```

- Validate
```bash
kubectl get deploy
```


# **Namespaces**

## **Kuberenetes and Namespace**
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

## **1. Create a new namespace**
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


## **2. List Namespaces**
```bash
kubectl get namespace
kubectl get ns
```

```bash
NAME              STATUS   AGE
default           Active   1d
kube-node-lease   Active   1d
kube-public       Active   1d
kube-system       Active   1d
```



## **3. Setting the namespace for a request**
When we run it request it runs for the **default namespace**. To set the namespace for a current request, use the `--namespace` flag or shorthand `-n`.

```bash
kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>

kubectl get pods --namespace=<insert-namespace-name-here>

kubectl get pods -n=<insert-namespace-name-here>
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



## **4. Setting Request for all namespaces**

To set the request for all namespaces, use the `--all-namespaces` flag or shorthand `-A`.

```bash
kubectl get pods --all-namespaces

kubectl get pods -A
```


## **5. Setting the namespace preference**
You can permanently save the namespace for all subsequent kubectl commands in that context.

<div style="text-align: right"> <span style="color:blue"> *Shorthand "-n" not allowed </span> </div>

```bash
kubectl config set-context --current --namespace=<insert-namespace-name-here>

# Validate it
kubectl config view --minify | grep namespace:
```

## **Not All Objects are in a Namespace** 
Most Kubernetes resources (e.g. pods, services, replication controllers, and others) are in some namespaces. However namespace resources are not themselves in a namespace. And low-level resources, such as **nodes and persistentVolumes, are *not in any namespace***.

To see which Kubernetes resources are and aren't in a namespace:

```bash
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

# **Resource Quota**

A resource quota, defined by a `ResourceQuota` object, provides constraints that _limit aggregate resource consumption per namespace_. 

It can limit the _quantity of objects_ that can be created in a namespace by type, as well as the _total amount of compute resources_ that may be consumed by resources in that namespace.

- Users create resources (pods, services, etc.) in the namespace, and the quota system tracks usage to ensure it does not exceed hard resource limits defined in a ResourceQuota.

- If creating or updating a resource violates a quota constraint, the request will fail with HTTP status code 403 FORBIDDEN with a message explaining the constraint that would have been violated.

- If quota is enabled in a namespace for compute resources like cpu and memory, users must specify requests or limits for those values; otherwise, the quota system may reject pod creation. 
  - **Hint**: Use the `LimitRanger` admission controller to force defaults for pods that make no compute resource requirements.

## **Resource Quota Definition File**

### **Compute Quota**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
  namespace: dev # optional
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    requests.nvidia.com/gpu: 4
```
### **Count Quota**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: object-counts
  namespace: dev # optional
spec:
  hard:
    configmaps: "10"
    persistentvolumeclaims: "4"
    pods: "4"
    replicationcontrollers: "20"
    secrets: "10"
    services: "10"
    services.loadbalancers: "2"
```

```bash
kubectl create -f <quota definition file path>
```

## **Imperative Commands**
```bash
kubectl create quota test --hard=count/deployments.apps=2,count/replicasets.apps=4,count/pods=3,count/secrets=4 --namespace=myspace
```

## **Compute Resource Quota**

You can limit the total sum of [compute resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) that can be requested in a given namespace.

The following resource types are supported:

| Resource Name | Description |
| --------------------- | ----------------------------------------------------------- |
| `limits.cpu` | Across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value. |
| `limits.memory` | Across all pods in a non-terminal state, the sum of memory limits cannot exceed this value. |
| `requests.cpu` | Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value. |
| `requests.memory` | Across all pods in a non-terminal state, the sum of memory requests cannot exceed this value. |
| `hugepages-<size>` | Across all pods in a non-terminal state, the number of huge page requests of the specified size cannot exceed this value. |
| `cpu` | Same as `requests.cpu` |
| `memory` | Same as `requests.memory` |

## **Storage Resource Quota**

You can limit the total sum of [storage resources](/docs/concepts/storage/persistent-volumes/) that can be requested in a given namespace.

In addition, you can limit consumption of storage resources based on associated storage-class.

| Resource Name | Description |
| --------------------- | ----------------------------------------------------------- |
| `requests.storage` | Across all persistent volume claims, the sum of storage requests cannot exceed this value. |
| `persistentvolumeclaims` | The total number of [PersistentVolumeClaims](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) that can exist in the namespace. |
| `<storage-class-name>.storageclass.storage.k8s.io/requests.storage` | Across all persistent volume claims associated with the `<storage-class-name>`, the sum of storage requests cannot exceed this value. |
| `<storage-class-name>.storageclass.storage.k8s.io/persistentvolumeclaims` | Across all persistent volume claims associated with the storage-class-name, the total number of [persistent volume claims](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) that can exist in the namespace. |

For example, if an operator wants to quota storage with `gold` storage class separate from `bronze` storage class, the operator can
define a quota as follows:

* `gold.storageclass.storage.k8s.io/requests.storage: 500Gi`
* `bronze.storageclass.storage.k8s.io/requests.storage: 100Gi`

## **Object Count Quota**

You can set quota for the total number of certain resources of all standard,
namespaced resource types using the following syntax:

* `count/<resource>.<group>` for resources from non-core groups
* `count/<resource>` for resources from the core group

Here is an example set of resources users may want to put under object count quota:

* `count/persistentvolumeclaims`
* `count/services`
* `count/secrets`
* `count/configmaps`
* `count/replicationcontrollers`
* `count/deployments.apps`
* `count/replicasets.apps`
* `count/statefulsets.apps`
* `count/jobs.batch`
* `count/cronjobs.batch`

The same syntax can be used for custom resources.
For example, to create a quota on a `widgets` custom resource in the `example.com` API group, use `count/widgets.example.com`.

When using `count/*` resource quota, an object is charged against the quota if it exists in server storage.
These types of quotas are useful to protect against exhaustion of storage resources.  For example, you may
want to limit the number of Secrets in a server given their large size. Too many Secrets in a cluster can
actually prevent servers and controllers from starting. You can set a quota for Jobs to protect against
a poorly configured CronJob. CronJobs that create too many Jobs in a namespace can lead to a denial of service.
