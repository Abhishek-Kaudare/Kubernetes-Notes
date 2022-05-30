
- [**Overide CMD and ENTRYPOINT in Pod Declaration**](#overide-cmd-and-entrypoint-in-pod-declaration)
- [**Environment Variables**](#environment-variables)
  - [**Overide ENV in commandline**](#overide-env-in-commandline)
  - [**Pod Configuration**](#pod-configuration)
  - [**Config Maps**](#config-maps)
    - [**Create Config map using Imperative Commands**](#create-config-map-using-imperative-commands)
      - [**Key Value Pair in CLI**](#key-value-pair-in-cli)
      - [**Key Value Pair in file**](#key-value-pair-in-file)
    - [**Yaml File**](#yaml-file)
    - [**Using Configmaps in Pods**](#using-configmaps-in-pods)
      - [**Using Configmap**](#using-configmap)
      - [**Using Configmap for specific key**](#using-configmap-for-specific-key)
      - [**Using Configmap from volumes**](#using-configmap-from-volumes)
- [**Secrets**](#secrets)
  - [**Create Secret using Imperative Commands**](#create-secret-using-imperative-commands)
    - [**Key Value Pair in CLI**](#key-value-pair-in-cli-1)
    - [**Key Value Pair in file**](#key-value-pair-in-file-1)
  - [**Create Secret using Declarative File**](#create-secret-using-declarative-file)
    - [**Encode text in linux**](#encode-text-in-linux)
    - [**Decode text in linux**](#decode-text-in-linux)
  - [**Using Secrets in Pods**](#using-secrets-in-pods)
    - [**Using Secret**](#using-secret)
    - [**Using Secret for specific key**](#using-secret-for-specific-key)
    - [**Using Secret from volumes**](#using-secret-from-volumes)
  - [**Other Commands**](#other-commands)
    - [**Get Secrets**](#get-secrets)
    - [**Describe Secrets**](#describe-secrets)
    - [**Describe Secrets with values**](#describe-secrets-with-values)
- [**Security Context**](#security-context)
- [**Service Accounts**](#service-accounts)
  - [**Service Account Token**](#service-account-token)
    - [**Application on K8's Cluster**](#application-on-k8s-cluster)
      - [**Default Service Account**](#default-service-account)
      - [**Different Service Account**](#different-service-account)
- [**Resource Requirements**](#resource-requirements)
  - [**CPU**](#cpu)
  - [**Memory**](#memory)
- [**Taints & Tolerations**](#taints--tolerations)
  - [**Untaint Node**](#untaint-node)
- [**Node Selectors**](#node-selectors)
  - [**Add label to Node**](#add-label-to-node)
  - [**Creating a pod with node selectors**](#creating-a-pod-with-node-selectors)
  - [**Limits of Node Selectors**](#limits-of-node-selectors)
- [Affinity and anti-affinity](#affinity-and-anti-affinity)
  - [**Node Affinity**](#node-affinity)
    - [**Node affinity weight**](#node-affinity-weight)







# **Overide CMD and ENTRYPOINT in Pod Declaration** 


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      command: ["sleep"] # --entrypoint flag of docker
      args: ["10"] # --cmd of docker
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
```


# **Environment Variables**
## **Overide ENV in commandline** 
```bash
docker run node_launch -e APP_COLOR=pink
```
## **Pod Configuration**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      env: # for configMap and secret use envFrom
        - name: APP_COLOR  
          value: pink  # for configMap and secret use valueFrom
```
## **Config Maps**
### **Create Config map using Imperative Commands**
#### **Key Value Pair in CLI**
```bash
kubectl create configmap <Config Map Name> \
                          --from-literal <Name1>=<Value1> \
                          --from-literal <Name2>=<Value2>
```
#### **Key Value Pair in file**
```bash
kubectl create configmap <Config Map Name> \
                          --from-file <file name> 
```
### **Yaml File**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <Config Map Name> # app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```
### **Using Configmaps in Pods**
#### **Using Configmap**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      envFrom:
      - configMapRef:   
          name: <Config Map Name>  # app-config
```

#### **Using Configmap for specific key**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      env:
      - name: <Env Name> # APP_COLOR
          valueFrom:
          - configMapKeyRef:   
              name: <Config Map Name>  # app-config
              key: <Key Name>  # APP_COLOR
```

#### **Using Configmap from volumes**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      volumes:
      - name: <Volume Name> 
        configMap:   
          name: <Config Map Name>
```

# **Secrets**
## **Create Secret using Imperative Commands**

### **Key Value Pair in CLI**
```bash
kubectl create secret generic <Secret Name> \
                          --from-literal <Name1>=<Value1> \
                          --from-literal <Name2>=<Value2>
```
### **Key Value Pair in file**
```bash
kubectl create secret generic <Secret Name> \
                          --from-file <file name> 
```
## **Create Secret using Declarative File**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: <Secret Name> # app-secret
data:
  DB_Host: bX1zcWw= # encode(mysql)
  DB_User: cm9vdA== # encode(root)
  DB_Password: cGFzd3Jk= # encode(paswrd)
```
### **Encode text in linux**
```bash
echo -n '<text>' | base64

# example
echo -n 'mysql' | base64
# Output
bX1zcWw=
```

### **Decode text in linux**
```bash
echo -n '<text>' | base64 --decode

# example
echo -n 'bX1zcWw=' | base64 --decode
# Output
mysql
```

## **Using Secrets in Pods**

### **Using Secret**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      envFrom:
      - secretMapRef:   
          name: <Secret Name>  # app-secret
```

### **Using Secret for specific key**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      env:
      - name: <Env Name> # DB_Host
          valueFrom:
          - secretKeyRef:   
              name: <Secret Name>  # app-secret
              key: <Key Name>  # DB_Host
```

### **Using Secret from volumes**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
    - name: myapp
      image: ubuntu
      volumes:
      - name: <Volume Name> 
        secret:   
          secretName: <Secret Name>  # app-secret
```


!!! note **More on Secrets**
    
    Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.

    The concept of safety of the Secrets is a bit confusing in Kubernetes. The [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret) page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it.

    Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

    - Not checking-in secret object definition files to source code repositories.
    - [Enabling Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets so they are stored encrypted in ETCD.

    Also the way kubernetes handles secrets. Such as:

    - A secret is only sent to a node if a pod on that node requires it.
    - Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
    - Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

    Read about the [protections](https://kubernetes.io/docs/concepts/configuration/secret/#protections) and [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks) of using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#risks)

    Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, [HashiCorp Vault](https://www.vaultproject.io/). I hope to make a lecture on these in the future.

## **Other Commands**
### **Get Secrets**
```bash
kubectl get secrets
# Output
```

### **Describe Secrets**
```bash
kubectl describe secrets
# Output
```


### **Describe Secrets with values**
```bash
kubectl get secret <secret name> -o yaml
# Output
```

# **Security Context**

When you run a `docker container you have the option to define a set of security standards` such as the id of the user used to run the container, the Linux capabilities that can be added or removed from the container etc.

`These can be configured in Kubernetes as well.`

As you know already in Kubernetes, _containers are encapsulated in pods._

- You may choose to configure the security settings at a container level or at a pod level.
  
- If you configure it at a pod level, the settings will carry over to all the containers within the pod.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  # Adds security context for the pod
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
```
- If you configure it at container level then it will only apply for that container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      # Adds security context for the container
      securityContext:
        runAsUser: 1000
        # Only supported at the container level NOT at the pod level
        capabilities:
          add: ["MAC_ADMIN"]
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
```
- If you configure it at both the pod and the container, the settings on the container will override the settings on the pod.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  # Adds security context for the pod
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      # Adds security context for the container and overrides pod
      securityContext:
        runAsUser: 1000
        # Only supported at the container level NOT at the pod level
        capabilities:
          add: ["MAC_ADMIN"]
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
```

!!! danger ""
    `Capabilities` supported at the container level NOT at the pod level.

    
- Get the user of pod in current namespace

```bash
kubectl exec <pod-name> -- whoami
```

# **Service Accounts**
!!!
    A service account named `Default` is automatically created.
    Each namespace has its own default service account whenever a part is created.

The concept of service accounts is linked to other security related concepts and coordinates, such as authentication authorization, role based access controls, etc..

There are two types of accounts in Kubernetes, an user account and a service account. `User Accounts` are used by users and `Service Accounts` are used by machines.

- Create Service Accounts

```bash
kubectl create serviceaccount <sa-name>
```

- List Service Accounts
```bash
kubectl get serviceaccount
```

- Describe Service Accounts
```bash
kubectl describe serviceaccount <sa-name>
-----------------------------------------
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   default-token-8qrrx
Tokens:              default-token-8qrrx
Events:              <none>
```
## **Service Account Token**
When the service account is created, it also creates a token automatically, the `service account token` is what must be used by the external application while authenticating to the Kubernetes API.

The token is `stored as a secret` object.

When a service account is created, 
1. It first creates the service account object.
2. Then it generates a token for the service account.
3. It then creates a secret object and stores that token inside the secret object.
4. The secret object is then linked to the service account under `Tokens` key.

Describe the secret named under `Tokens` in SA to get the Token value. This token can be used as **_Authentication Bearer Token_** while making request to Kubernetes Api.

Example,
```bash
curl https://192.168.56.70:6443/api -insecure --header "Authentication Bearer <token-value>"
```

### **Application on K8's Cluster**

If we can have our custom Kubernetes dashboard application or the Prometheus application deployed on the Kubernetes cluster itself.

In that case, this whole process of exporting the service account token and configuring the third party application to use it can be made simple by automatically mounting the service token secret as a volume inside the pod, hosting the third party application.

That way, the token to access the Kubernetes API is already placed inside the port and can be easily read by the application.

#### **Default Service Account**
    - If you go back and look at the list of service accounts, you will see that there is a default service account that exists already for every namespace in Kubernetes.
    - The default service account and its token are automatically mounted to that pod as a volume mount.

For example, we have a simple pod definition file that creates a pod using my custom Kubernetes dashboard image.

We haven't specified any secrets or elements in the definition file.

However, when the pod is created, if you look at the details of the pod by running the kubectl describe pod command, 
```bash
kubectl describe po <pod-name>
------------------------------
Name:         <pod-name>
Namespace:    default
Priority:     0
...
Containers:
  web-dashboard:
    Image:          image
    Mounts:         /var/run/secrets/kubernetes.io/serviceaccount from default-token-k8wjl (ro)
    ....
```

you see that if volume is automatically created from the secret named default token, which is in fact the secret containing the token for this default service account, the secret

!!!
    Token is mounted at location `/var/run/secrets/kubernetes.io/serviceaccount` inside the pod.

So from inside the pod, if you run the `ls` command to list the contents of the directory, you will see the Secret Mountain as three separate files.

```bash
kubectl exec -t <pod-name> -- ls /var/run/secrets/kubernetes.io/serviceaccount
---------------------------------------------------------------------------
ca.crt
namespace
token
```
The one with the actual token is the file named token. If you view contents of that file, you will see the token to be used for accessing the Kubernetes API.

Default service account is very much restricted. It only has permission to run basic common API queries.

#### **Different Service Account**

If you'd like to use a different service account, modify the definition file to include a service account field and specify the name of the new service account.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  serviceAccount: my-service-account
  containers:
    - name: myapp
      image: myapp
```

- We cannot edit the service account of an existing pod. 
- You must delete and recreate the pod. 
- However, in case of a deployment, you will be able to edit the service account as any changes to the definition file will automatically trigger a new rollout for the deployment. 
- So the deployment will take care of deleting and recreating new parts with the right service account.

So remember, k8 automatically mounts the default service account if you haven't explicitly specified any. You may choose not to mount a service account automatically by setting the `autoMountServiceAccountToken` field to `false`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  automountServiceAccountToken: false
  containers:
    - name: myapp
      image: myapp
```
# **Resource Requirements**

Whenever a pod is placed on a node, it consumes resources available to that node.

Scheduler decides which node a pod goes to. The scheduler takes into consideration the amount of resources required by a pod and those available on the nodes.

If the node has no sufficient resources, the scheduler avoids placing the part on that node, instead places the part on one where sufficient resources are available.

If there is no sufficient resources available on any of the nodes, k8 holds back scheduling the pod. You will see the pod in a `pending` state. If you look at the events, you will see the reason `insufficient <Resource>`.

By default the k8s assumes resurces for containers within pods to require `0.5 CPU and 256 Mi of memory`


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "500Mi"
          cpu: 1
```
## **CPU**
One count of C.P.U is equivalent to one vCPU, 

    1 AWS vCPU
    1 GCP Core
    1 Azure Core
    1 Hyperthread

`Min value` of CPU can be `0.1 ~ 100m` and can go lower to `1m`

## **Memory**

Unit of measure

    Kilobyte (kB) = 1,000 bytes

    Megabyte (MB) = 1,000,000 bytes

    Gigabyte (GB) = 1,000,000,000 bytes


    Kibibyte (KiB) = 1,024 bytes

    Mebibyte (MiB) = 1,048,576 bytes

    Gibibyte (GiB) = 1,073,741,824 bytes

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "500Mi"
          cpu: 1
        limits:
          memory: "1Gi"
          cpu: 2
```
!!! Danger Above limits the K8s scheduler
     
    CPU - Throttle

    Memory - Terminate (Will mostly have state terminated and reason OOMKilled)

!!! Note Notes on default resource requirements and limits
    In the previous lecture, I said - "When a pod is created the containers are assigned a default CPU request of .5 and memory of 256Mi". For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.


    ```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: mem-limit-range
    spec:
      limits:
      - default:
          memory: 512Mi
        defaultRequest:
          memory: 256Mi
        type: Container
    ```

    https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

    ```yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: cpu-limit-range
    spec:
      limits:
      - default:
          cpu: 1
        defaultRequest:
          cpu: 0.5
        type: Container
    ```

    https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/

    References:

    https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource


# **Taints & Tolerations**

`kubectl taint nodes <node-name> key=value:taint-effect`
taint-effect
- NoSchedule
- PreferNoSchedule
- NoExecute (Existing pod which can't tolerate will be evicted i.e. terminated)

```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
  # Toleration values should map to a node taint.
  tolerations:
    # Values must be encoded in double quotes.
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

Taints and tolerations are only meant to restrict nodes from excepting certain pods. Taints and tolerations does not tell the pod to go to a particular node. Instead it tells the node to only accept pods with certain tolerations.

So a pod with certain toleration may go to other node with no taint.

!!! Notes on taints on kubemaster

    **The Scheduler does not shedule any pods on the master node. Why is that?**
    
    When the Kubernetes cluster is first set up, a taint is set on the master node. Automatically that prevents any pods from being scheduled on this node.You can see this as well as modify this behavior if required.

    `However a best practice is not to deploy application workloads on a master server.` 
    
    ```bash
    kubectl describe node controlplane | grep Taints
    ------------------------------------------------
    Taints:             node-role.kubernetes.io/master:NoSchedule
    ```

## **Untaint Node**

To Untaint a node just add `-` at the end without the space.
```bash
kubectl taint nodes <node-name> key=value:taint-effect-
```

# **Node Selectors**

To set a pod on certain node with some characteristics we can use node selectors.

Node selectors are just some labels that needs to be matched between node and pod.

## **Add label to Node**

```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```
Example

```bash
kubectl label nodes node01 size=Large
```

## **Creating a pod with node selectors**

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

  nodeSelector:
      size: Large
```

## **Limits of Node Selectors**
We are just using one label so this is not enough for complex requirement, like if we want to place the node on `medium or large nodes`, or say on `all node that are not small`.

# Affinity and anti-affinity

`nodeSelector` is the simplest way to constrain Pods to nodes with specific labels. _Affinity and anti-affinity expands the types of constraints_ you can define. Some of the benefits of affinity and anti-affinity include:

- The _affinity/anti-affinity_ language is more expressive. `nodeSelector` only selects nodes with all the specified labels. _Affinity/anti-affinity_ gives you more control over the selection logic.
  
- You can _indicate that a rule is soft or preferred_, so that the scheduler still schedules the Pod even if it can't find a matching node.
  
- You can _constrain a Pod using labels on other Pods running on the node_ (or other topological domain), instead of just node labels, which allows you to _define rules for which Pods can be co-located on a node_.
  

The affinity feature consists of two types of affinity:

- `Node affinity` functions like the `nodeSelector` field but is more expressive and allows you to specify soft rules.
- `Inter-pod affinity/anti-affinity` allows you to constrain Pods against labels on other Pods.
  
## **Node Affinity**

Node affinity is conceptually similar to `nodeSelector`, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

`requiredDuringSchedulingIgnoredDuringExecution`: The scheduler can't schedule the Pod unless the rule is met. This functions like `nodeSelector`, but with a more expressive syntax.

`preferredDuringSchedulingIgnoredDuringExecution`: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

> **Note:** 
> 
> In the preceding types, IgnoredDuringExecution means that if the node labels change after Kubernetes schedules the Pod, the Pod continues to run.

You can specify node affinities using the `.spec.affinity.nodeAffinity` field in your Pod spec.

For example, consider the following Pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```
In this example, the following rules apply:

The node *must* have a label with the key `kubernetes.io/os` and the value `linux`.

The node *preferably* has a label with the key `another-node-label-key` and the value `another-node-label-value`.

You can use the `operator` field to specify a logical operator for Kubernetes to use when interpreting the rules. You can use `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt` and `Lt`.

`NotIn` and `DoesNotExist` allow you to define _node anti-affinity_ behavior. Alternatively, you can use node taints to repel Pods from specific nodes.

> **Note:**
>
> If you specify both `nodeSelector` and `nodeAffinity`, *both* must be satisfied for the Pod to be scheduled onto a node.
>
> If you specify multiple `nodeSelectorTerms` associated with `nodeAffinity` types, then the Pod can be scheduled onto a node if one of the specified `nodeSelectorTerms` can be satisfied.
> 
> If you specify multiple `matchExpressions` associated with a single `nodeSelectorTerms`, then the Pod can be scheduled onto a node only if all the `matchExpressions` are satisfied.

See [Assign Pods to Nodes using Node Affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/) for more information.

### **Node affinity weight**

You can specify a `weight` between 1 and 100 for each instance of the `preferredDuringSchedulingIgnoredDuringExecution` affinity type. When the scheduler finds nodes that meet all the other scheduling requirements of the Pod, the scheduler iterates through every preferred rule that the node satisfies and adds the value of the `weight` for that expression to a sum.

The final sum is added to the score of other priority functions for the node. Nodes with the highest total score are prioritized when the scheduler makes a scheduling decision for the Pod.

For example, consider the following Pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-affinity-anti-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: label-1
            operator: In
            values:
            - key-1
      - weight: 50
        preference:
          matchExpressions:
          - key: label-2
            operator: In
            values:
            - key-2
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```
If there are two possible nodes that match the `requiredDuringSchedulingIgnoredDuringExecution` rule, one with the `label-1:key-1` label and another with the `label-2:key-2` label, the scheduler considers the weight of each node and adds the weight to the other scores for that node, and schedules the Pod onto the node with the highest final score.


> **Note:** 
> 
> If you want Kubernetes to successfully schedule the Pods in this example, you must have existing nodes with the `kubernetes.io/os=linux` label.

