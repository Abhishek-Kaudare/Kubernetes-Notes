Where is the default kubeconfig file located in the current environment?

-- /root/.kube/config

KubeConfig File Example:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: data==
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: data==
    client-key-data: data==
```

I would like to use the dev-user to access test-cluster-1. Set the current context to the right one so I can do that.
```bash
k config use-context research --kubeconfig=/root/my-kube-config
kubectl config --kubeconfig=/root/my-kube-config current-context
```

We don't want to have to specify the kubeconfig file option on each command. Make the my-kube-config file the default kubeconfig.

```bash
cp my-kube-config .kube/config
```

With the `current-context` set to research, we are trying to access the cluster. However something seems to be wrong. Identify and fix the issue.

Try running the `kubectl get pods` command and look for the error. All users certificates are stored at `/etc/kubernetes/pki/users`.