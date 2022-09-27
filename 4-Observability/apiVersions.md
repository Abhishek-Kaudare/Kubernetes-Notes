# **Api Versions**

In Kubernetes versions : X.Y.Z
Where X stands for major, Y stands for minor and Z stands for patch version.

## **Install a plugin (kubectl convert)**
### **Download the package**
```
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
```

### **Change permissions to move it to the binaries**
```
chmod +x kubectl-convert
mv kubectl-convert /usr/local/bin/kubectl-convert
```

### **Check if successful**
```
kubectl-convert --version
```

### **Use it to change output version of an ingress file**
```
kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml
```

### **Check if it was successful**
```
kubectl apply -f ingress-new.yaml &
kubectl get ing ingress-space -o yaml | grep apiVersion
```

## **Upgrade the ApiGroup of YAML to new version**
With help of the kubectl convert command, change the deprecated API version to the `apiGroup/newVersion`.

Example:
```bash
kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml

# Create the resource 
kubectl create -f ingress-new.yaml

# Check Version
kubectl get ing ingress-space -o yaml | grep apiVersion
```


## **Identify which API group a resource**

Run the command `kubectl explain job` and see the `API Version` in the top of the line.

At first will be the `API` group and second will be the `version`: `<group>/<version>`

## **What is the preferred version for resource**

To identify the preferred version, run the following commands as follows:

```bash
kubectl proxy 8001&
curl localhost:8001/apis/<api>
```

## **Enable the `version` version for `apiGroup` on the controlplane node.**

As a good practice, take a backup of that apiserver manifest file before going to make any changes.
In case, if anything happens due to misconfiguration you can replace it with the backup file.

```bash
cp -v /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.backup
```

Now, open up the kube-apiserver manifest file in the editor of your choice. It could be vim or nano.

```bash
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add the `--runtime-config` flag in the `command` field as follows:
```bash
- command:
    - kube-apiserver
    - --advertise-address=10.18.17.8
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --runtime-config=rbac.authorization.k8s.io/v1alpha1 # --> This one
```

After that `kubelet` will detect the new changes and will recreate the `apiserver` pod.

It may take some time.

```bash
kubectl get po -n kube-system
```