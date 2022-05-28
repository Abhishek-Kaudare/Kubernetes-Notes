
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



