- [DaemonSet](#daemonset)
- [What's the difference between a Service and a Deployment in Kubernetes?](#whats-the-difference-between-a-service-and-a-deployment-in-kubernetes)
- [Pod restart options](#pod-restart-options)
- [Find processes](#find-processes)
- [Find and delete static pod](#find-and-delete-static-pod)
  - [ssh to node:](#ssh-to-node)
  - [find kubelet config file from this](#find-kubelet-config-file-from-this)
  - [found the file](#found-the-file)
  - [now find where staticpods are located](#now-find-where-staticpods-are-located)
- [check the path and find the filename with .yaml extension](#check-the-path-and-find-the-filename-with-yaml-extension)
- [Custom Scheduler](#custom-scheduler)
- [Edit Deployments](#edit-deployments)
  - [first edit the running deployment](#first-edit-the-running-deployment)
- [InitContainers](#initcontainers)
- [Self Healing Applications](#self-healing-applications)
- [Cordone and Drain](#cordone-and-drain)
- [Upgrade K8S Cluster](#upgrade-k8s-cluster)
  - [Drain the master node](#drain-the-master-node)
  - [Update package manager in master node](#update-package-manager-in-master-node)
  - [Update kubeadm in master node](#update-kubeadm-in-master-node)
  - [Apply update kubeadm in master node](#apply-update-kubeadm-in-master-node)
  - [Upgrade Kubelet in master node](#upgrade-kubelet-in-master-node)
  - [Do the same for worker node](#do-the-same-for-worker-node)
- [Backup and Restore](#backup-and-restore)
  - [Save a snapshot](#save-a-snapshot)
  - [Stop the api-server](#stop-the-api-server)
  - [Restore the backed up etcd the api-server](#restore-the-backed-up-etcd-the-api-server)
  - [reload the daemon](#reload-the-daemon)
  - [Restart the etcd service](#restart-the-etcd-service)
  - [Restart the api-server](#restart-the-api-server)
  - [Get etcd address to reach from namespace kube-system and node master/controlplane](#get-etcd-address-to-reach-from-namespace-kube-system-and-node-mastercontrolplane)
  - [etcd, Controller Manager, Kube API Server, Kube Scheduler config file location](#etcd-controller-manager-kube-api-server-kube-scheduler-config-file-location)
- [Article on Setting up Basic Authentication](#article-on-setting-up-basic-authentication)
  - [Setup basic authentication on Kubernetes (Deprecated in 1.19)](#setup-basic-authentication-on-kubernetes-deprecated-in-119)
  - [Create a file with user details locally at /tmp/users/user-details.csv](#create-a-file-with-user-details-locally-at-tmpusersuser-detailscsv)
    - [User File Contents](#user-file-contents)
  - [Modify the kube-apiserver startup options to include the `basic-auth` file](#modify-the-kube-apiserver-startup-options-to-include-the-basic-auth-file)
  - [Create the necessary roles and role bindings for these users:](#create-the-necessary-roles-and-role-bindings-for-these-users)
  - [This role binding allows "jane" to read pods in the "default" namespace.](#this-role-binding-allows-jane-to-read-pods-in-the-default-namespace)
  - [Once created, you may authenticate into the kube-api server using the users credentials](#once-created-you-may-authenticate-into-the-kube-api-server-using-the-users-credentials)
- [TLS Security](#tls-security)
- [Deployment rollbacks](#deployment-rollbacks)
- [A quick note about Secrets!](#a-quick-note-about-secrets)
    - [to convert a secret to base64](#to-convert-a-secret-to-base64)
- [kubectl replace vs kubectl apply](#kubectl-replace-vs-kubectl-apply)
- [Assign Nodes for pods](#assign-nodes-for-pods)
- [Tips](#tips)
  - [debug connectivity within service](#debug-connectivity-within-service)
- [Taint/Toleration/Affinity](#tainttolerationaffinity)
- [Admission Controllers](#admission-controllers)
  - [Which plugins are enabled by default?](#which-plugins-are-enabled-by-default)
  - [Important Errors](#important-errors)
  - [Why pv and pvc isn't enough](#why-pv-and-pvc-isnt-enough)
  - [Change runtime config for a resource](#change-runtime-config-for-a-resource)
    - [take backup](#take-backup)
    - [change file](#change-file)
    - [save the file and run this one to check if it was successful](#save-the-file-and-run-this-one-to-check-if-it-was-successful)
  - [Install a plugin (kubectl convert)](#install-a-plugin-kubectl-convert)
    - [download the package](#download-the-package)
    - [change permissions to move it to the binaries](#change-permissions-to-move-it-to-the-binaries)
    - [check if successful](#check-if-successful)
    - [use it to change output version of an ingress file](#use-it-to-change-output-version-of-an-ingress-file)
    - [check if it was successful](#check-if-it-was-successful)
  - [rolling update vs recreate](#rolling-update-vs-recreate)
    - [recreate](#recreate)
    - [rollingUpdate](#rollingupdate)
  - [CronJob vs Job](#cronjob-vs-job)
    - [Job is](#job-is)
    - [CronJob is job inside Cron schedule](#cronjob-is-job-inside-cron-schedule)
  - [Different kind of secrets](#different-kind-of-secrets)
    - [generic secret](#generic-secret)
    - [volume mount secret](#volume-mount-secret)
- [Useful resources:](#useful-resources)


## DaemonSet
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

Some typical uses of a `DaemonSet` are:

- running a cluster storage daemon on every node
- running a logs collection daemon on every node
- running a node monitoring daemon on every node

In a simple case, one `DaemonSet`, covering all nodes, would be used for each type of daemon. A more complex setup might use multiple `DaemonSet`s for a single type of daemon, but with different flags and/or different memory and cpu requests for different hardware types.

Writing a DaemonSet Spec
Create a DaemonSet
You can describe a DaemonSet in a YAML file. For example, the daemonset.yaml file below describes a DaemonSet that runs the fluentd-elasticsearch Docker image:

`controllers/daemonset.yaml` Copy `controllers/daemonset.yaml` to clipboard
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```
Create a DaemonSet based on the YAML file:
```
kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
```

## What's the difference between a Service and a Deployment in Kubernetes?
- A deployment is responsible for keeping a set of pods running.
- A service is responsible for enabling network access to a set of pods.
- We could use a deployment without a service to keep a set of identical pods running in the Kubernetes cluster. The deployment could be scaled up and down and pods could be replicated. Each pod could be accessed individually via direct network requests (rather than abstracting them behind a service), but keeping track of this for a lot of pods is difficult.
- We could also use a service without a deployment. We'd need to create each pod individually (rather than "all-at-once" like a deployment). Then our service could route network requests to those pods via selecting them based on their labels.
- Services and Deployments are different, but they work together nicely.

## Pod restart options
`Always` - means that the container will be restarted even if it exited with a zero exit code (i.e. successfully). This is useful when you don't care why the container exited, you just want to make sure that it is always running (e.g. a web server). This is the default.

`OnFailure` -  means that the container will only be restarted if it exited with a non-zero exit code (i.e. something went wrong). This is useful when you want accomplish a certain task with the pod, and ensure that it completes successfully - if it doesn't it will be restarted until it does.

`Never` - means that the container will not be restarted regardless of why it exited.


## Find processes
`-e` and `-f` are options to the ps command, and pipes take the output of one command and pass it as the input to another. Here is a full breakdown of this command:  
`ps` - list processes  
`-e` - show all processes, not just those belonging to the user  
`-f` - show processes in full format (more detailed than default)  
`-i` - show case insensitive matches   
`command 1 | command 2` - pass output of `command 1` as input to `command 2`
`grep` find lines containing a pattern
processname - the pattern for grep to search for in the output of `ps -ef`

So altogether,
```
ps -ef | grep processname
```
means: look for lines containing processname in a detailed overview/snapshot of all current processes, and display those lines

To find something in a file:  
```
grep -i staticpod /var/lib/kubelet/config.yaml
```

## Find and delete static pod
### ssh to node:  
```
ssh node01
```
### find kubelet config file from this
```
ps -ef | grep /usr/bin/kubelet
```

### found the file
`/var/lib/kubelet/config.yaml`

### now find where staticpods are located
```
grep -i staticpod /var/lib/kubelet/config.yaml
```

## check the path and find the filename with .yaml extension
```
rm /path/to/file
```

## Custom Scheduler
scheduler files are normally located in `/etc/kubernetes/manifests/`

## Edit Deployments
### first edit the running deployment
```
kubectl edit deployment frontend
```
then save and exit. If the strategy is rolling update then pods in the replicaset will be automatically be down one by one and new specified changes in the pod will be up one by one so that the application is up all the time

## InitContainers
In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.

But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only  one time when the pod is first created. Or a process that waits  for an external service or database to be up before the actual application starts. That's where initContainers comes in.

An `initContainer` is configured in a pod like all other containers, except that it is specified inside a initContainers section,  like this:

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts. 

You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run one at a time in sequential order.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

Read more about initContainers here. And try out the upcoming practice test.

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

## Self Healing Applications
Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.

Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes. However these are not required for the CKA exam and as such they are not covered here. These are topics for the Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.

## Cordone and Drain
If we drain a node, this is similar to the `NoExecute` taint on nodes, which means the Pods running on it will be evicted and those will be recreated on some other node, as the Pods are controlled by the Deployment. Please ensure not to `drain` while you have standalone Pods as those Pods would not be created as they are not controlled by other high level objects such as Deployments. You may have to use `-–force` with `kubectl drain` if you still wish to evict standalone Pods. However the standard way of deploying applications these days is using Deployments, and `kubectl drain` works perfectly with deployments.

The drain command wouldn’t work as is, cause it has got daemon set Pods running on them
```
networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl drain k8s-node3
node/k8s-node3 cordoned
error: unable to drain node "k8s-node3", aborting command...

There are pending nodes to be drained:
k8s-node3
error: cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/calico-node-z5w62, kube-system/kube-proxy-h5pp4
```
The associated `DaemonSets` are part of the `kube-system` namespace, and those would be launched when the cluster is launched, The calico daemon set is responsible for inter networking communication between objects in the cluster and `kube-proxy` is responsible exposing applications in or out of the cluster, In short both of these daemon sets are related to networking

We have to use the `–ignore-daemonsets` flag to drain the node now

```
networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl drain k8s-node3 --ignore-daemonsets
node/k8s-node3 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-z5w62, kube-system/kube-proxy-h5pp4
evicting pod "deploy7-5f5fff8f56-ljx9s"
evicting pod "deploy7-5f5fff8f56-v84qn"
evicting pod "deploy7-5f5fff8f56-qdbz4"
evicting pod "deploy7-5f5fff8f56-tvxl9"
pod/deploy7-5f5fff8f56-qdbz4 evicted
pod/deploy7-5f5fff8f56-tvxl9 evicted
pod/deploy7-5f5fff8f56-v84qn evicted
pod/deploy7-5f5fff8f56-ljx9s evicted
node/k8s-node3 evicted
```
The 10 replicas should now be distributed between k8s-node1 and k8s-node2
```
etworkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl get po -o wide | grep deploy7
deploy7-5f5fff8f56-7wx2r   1/1     Running   0          55s   192.168.1.116   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-grjj7   1/1     Running   0          14m   192.168.2.106   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-mg65p   1/1     Running   0          55s   192.168.2.108   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-np4kg   1/1     Running   0          55s   192.168.2.109   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-qjxjb   1/1     Running   0          14m   192.168.1.115   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-sznl4   1/1     Running   0          14m   192.168.2.105   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-vzw8q   1/1     Running   0          55s   192.168.1.117   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-wj79j   1/1     Running   0          14m   192.168.1.114   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-x6nrg   1/1     Running   0          14m   192.168.1.113   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-xgc8l   1/1     Running   0          14m   192.168.2.107   k8s-node2   <none>           <none>
```

Let’s see the cordon option now, This is only going to prevent any extra Pods from being added to the node, however the existing Pods are still going to be there, this is similar to the NoSchedule taint on nodes. Hence, its perfectly fine if we have any standalone pods on nodes that are going to be cordoned, and we don’t have use the `–ignore-daemonsets` flag here

Let’s cordon `k8s-node2`, there should be no change in the `kubect get po -o wide` output
```
etworkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl cordon k8s-node2
node/k8s-node2 cordoned

networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl get po -o wide | grep deploy7
deploy7-5f5fff8f56-7wx2r   1/1     Running   0          5m37s   192.168.1.116   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-grjj7   1/1     Running   0          19m     192.168.2.106   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-mg65p   1/1     Running   0          5m37s   192.168.2.108   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-np4kg   1/1     Running   0          5m37s   192.168.2.109   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-qjxjb   1/1     Running   0          19m     192.168.1.115   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-sznl4   1/1     Running   0          19m     192.168.2.105   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-vzw8q   1/1     Running   0          5m37s   192.168.1.117   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-wj79j   1/1     Running   0          19m     192.168.1.114   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-x6nrg   1/1     Running   0          19m     192.168.1.113   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-xgc8l   1/1     Running   0          19m     192.168.2.107   k8s-node2   <none>           <none>
```
Let’s delete the deployment and relaunch it, all the Pod replicas could only be scheduled on `k8s-node1` as `k8s-node2` is cordoned, and `k8s-node3` is drained
```
networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl delete deploy deploy7
deployment.extensions "deploy7" deleted

networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl get deployment
No resources found.

networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl create -f ex7-deploy.yaml

deployment.apps/deploy7 created

networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl get po -o wide | grep deploy7
deploy7-5f5fff8f56-7rrhg   1/1     Running   0          33s   192.168.1.127   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-fdmgz   1/1     Running   0          33s   192.168.1.125   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-ffhps   1/1     Running   0          33s   192.168.1.119   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-h5z6d   1/1     Running   0          33s   192.168.1.121   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-jpr5l   1/1     Running   0          33s   192.168.1.122   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-kvx4v   1/1     Running   0          33s   192.168.1.118   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-l997s   1/1     Running   0          33s   192.168.1.124   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-wpq6p   1/1     Running   0          33s   192.168.1.123   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-xjqj9   1/1     Running   0          33s   192.168.1.120   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-z8l5w   1/1     Running   0          33s   192.168.1.126   k8s-node1   <none>           <none>
```
Alright let’s revert to the previous settings, let’s now delete the deployment again, ‘uncordon’ both the nodes, so that the Pods get launched on all 3 nodes. Note that ‘uncordon’ is used to remove both drains and cordons
```
networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl delete deploy deploy7
deployment.extensions "deploy7" deleted
```
```
networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl uncordon k8s-node2 k8s-node3
node/k8s-node2 uncordoned
node/k8s-node3 uncordoned
```
```
networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl create -f ex7-deploy.yaml 
deployment.apps/deploy7 created
networkandcode@k8s-master:~/tech/kubernetes/cka$ kubectl get pods -o wide | grep deploy7
deploy7-5f5fff8f56-4khlg   1/1     Running   0          91s   192.168.3.103   k8s-node3   <none>           <none>
deploy7-5f5fff8f56-8cqcj   1/1     Running   0          91s   192.168.1.130   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-9l88d   1/1     Running   0          91s   192.168.1.128   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-blh7f   1/1     Running   0          91s   192.168.2.110   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-dndtf   1/1     Running   0          91s   192.168.1.129   k8s-node1   <none>           <none>
deploy7-5f5fff8f56-fthpt   1/1     Running   0          91s   192.168.3.105   k8s-node3   <none>           <none>
deploy7-5f5fff8f56-m999r   1/1     Running   0          91s   192.168.2.112   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-p6qrw   1/1     Running   0          91s   192.168.3.102   k8s-node3   <none>           <none>
deploy7-5f5fff8f56-qm7g4   1/1     Running   0          91s   192.168.2.111   k8s-node2   <none>           <none>
deploy7-5f5fff8f56-s74gj   1/1     Running   0          91s   192.168.3.104   k8s-node3   <none>           <none>
```

## Upgrade K8S Cluster
### Drain the master node
```
kubectl drain master/controlplane --ignore-daemonsets
```
### Update package manager in master node
```
apt update
```
### Update kubeadm in master node
```
apt install kubeadm=1.19.0-00
```
### Apply update kubeadm in master node
```
kubeadm upgrade apply v1.19.0
```
### Upgrade Kubelet in master node
```
apt install kubelet=1.19.0-00
```

### Do the same for worker node
```
ssh node01
apt install kubeadm=1.19.0-00
kubeadm upgrade node
apt install kubelet=1.19.0-00
logout
kubectl uncordon node01
```

## Backup and Restore
- `etcdctl` is a command line client for `etcd`.
- In all our Kubernetes Hands-on labs, the `ETCD` key-value database is deployed as a static pod on the master. The version used is v3.
- To make use of `etcdctl` for tasks such as back up and restore, make sure that you set the `ETCDCTL_API` to `3`.
- You can do this by exporting the variable `ETCDCTL_API` prior to using the etcdctl client. This can be done as follows:
  ```
  export ETCDCTL_API=3
  ```
### Save a snapshot
```
etcdctl snapshot save snapshot.db \
    --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/etcd/ca.cert \
    --cert=/etc/etcd/etcd-server.cert \
    --key=/etc/etcd/etcd-server.key
```

or

```
ETCDCTL_API=3 etcdctl \
                    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
                    --key=/etc/kubernetes/pki/etcd/server.key \
                    --endpoints=https://127.0.0.1:2379 \
                    --cert=/etc/kubernetes/pki/etcd/server.crt \
                    snapshot save /opt/snapshot-pre-boot.db
```
### Stop the api-server
```
service kube-apiserver stop
```
### Restore the backed up etcd the api-server
```
etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup
```
or
```
ETCDCTL_API=3 etcdctl \
            --data-dir /var/lib/etcd-from-backup \
            snapshot restore /opt/snapshot-pre-boot.db
```
### reload the daemon
```
systemctl daemon-reload
```
### Restart the etcd service
```
service etcd restart
```
### Restart the api-server
```
service kube-apiserver start
```

### Get etcd address to reach from namespace kube-system and node master/controlplane
```
kubectl describe pod etcd-controlplane -n kube-system | grep -i listen-client-urls
```

### etcd, Controller Manager, Kube API Server, Kube Scheduler config file location
```
/etc/kubernetes/manifests/
```

## Article on Setting up Basic Authentication
### Setup basic authentication on Kubernetes (Deprecated in 1.19)
Note: This is not recommended in a production environment. This is only for learning purposes. Also note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases

Follow the below instructions to configure basic authentication in a kubeadm setup.

### Create a file with user details locally at /tmp/users/user-details.csv

#### User File Contents
```
password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
password123,user4,u0004
password123,user5,u0005
```

Edit the `kube-apiserver` static pod configured by kubeadm to pass in the user details. The file is located at `/etc/kubernetes/manifests/kube-apiserver.yaml`


```
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
      <content-hidden>
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
    volumeMounts:
    - mountPath: /tmp/users
      name: usr-details
      readOnly: true
  volumes:
  - hostPath:
      path: /tmp/users
      type: DirectoryOrCreate
    name: usr-details
```

### Modify the kube-apiserver startup options to include the `basic-auth` file


```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
      <content-hidden>
    - --basic-auth-file=/tmp/users/user-details.csv
```
### Create the necessary roles and role bindings for these users:
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
``` 

### This role binding allows "jane" to read pods in the "default" namespace.
```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

Once you create a role, you have to create a rolebinding with a user in mind so that the user can use that role to do stuff

### Once created, you may authenticate into the kube-api server using the users credentials
```
curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"
```

## TLS Security
Encrypt a key before sending
```
openssl genrsa -out my-bank.key 1024
```
Decrypt a key before receiving
```
openssl rsa -in my-bank.key -pubout > mybank.pem
```

## Deployment rollbacks
Updating a Deployment
Here are some handy examples related to updating a Kubernetes Deployment:

Creating a deployment, checking the rollout status and history:
In the example below, we will first create a simple deployment and inspect the rollout status and the rollout history:

 
```
master $ kubectl create deployment nginx --image=nginx:1.16
deployment.apps/nginx created

master $ kubectl rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 0 of 1 updated replicas are available...
deployment "nginx" successfully rolled out

master $ kubectl rollout history deployment nginx
deployment.extensions/nginx
REVISION CHANGE-CAUSE
1     <none>
```

Using the `–revision` flag:
Here the revision 1 is the first version where the deployment was created.

You can check the status of each revision individually by using the `–revision` flag:

```
master $ kubectl rollout history deployment nginx --revision=1
deployment.extensions/nginx with revision #1

Pod Template:
 Labels:    app=nginx    pod-template-hash=6454457cdb
 Containers:  nginx:  Image:   nginx:1.16
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>
master $ 
```

Using the –record flag:
You would have noticed that the `change-cause` field is empty in the rollout history output. We can use the `–record` flag to save the command used to create/update a deployment against the revision number.
```
master $ kubectl set image deployment nginx nginx=nginx:1.17 --record
deployment.extensions/nginx image updated

master $ kubectl rollout history deployment nginx
deployment.extensions/nginx

REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
```

You can now see that the `change-cause` is recorded for the revision 2 of this deployment.

Lets make some more changes. In the example below, we are editing the deployment and changing the image from `nginx:1.17` to `nginx:latest` while making use of the `–record` flag.
```
master $ kubectl edit deployments. nginx --record
deployment.extensions/nginx edited

master $ kubectl rollout history deployment nginx
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
3     kubectl edit deployments. nginx --record=true



master $ kubectl rollout history deployment nginx --revision=3
deployment.extensions/nginx with revision #3

Pod Template: Labels:    app=nginx
    pod-template-hash=df6487dc Annotations: kubernetes.io/change-cause: kubectl edit deployments. nginx --record=true

 Containers:
  nginx:
  Image:   nginx:latest
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>

master $
 ```

Undo a change:
Lets now rollback to the previous revision:
```
master $ kubectl rollout undo deployment nginx
deployment.extensions/nginx rolled back

master $ kubectl rollout history deployment nginx
deployment.extensions/nginxREVISION CHANGE-CAUSE
1     <none>
3     kubectl edit deployments. nginx --record=true
4     kubectl set image deployment nginx nginx=nginx:1.17 --record=true




master $ kubectl rollout history deployment nginx --revision=4
deployment.extensions/nginx with revision #4Pod Template:
 Labels:    app=nginx    pod-template-hash=b99b98f9
 Annotations: kubernetes.io/change-cause: kubectl set image deployment nginx nginx=nginx:1.17 --record=true
 Containers:
  nginx:
  Image:   nginx:1.17
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>


master $ kubectl describe deployments. nginx | grep -i image:
  Image:    nginx:1.17
master $
```

With this, we have rolled back to the previous version of the deployment with the `image = nginx:1.17`.

## A quick note about Secrets!
Remember that secrets encode data in base64 format. Anyone with the `base64` encoded secret can easily decode it. As such the secrets can be considered as not very safe.

The concept of safety of the Secrets is a bit confusing in Kubernetes. The kubernetes documentation page and a lot of blogs out there refer to secrets as a “safer option” to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it’s not the secret itself that is safe, it is the practices around it.

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

Not checking-in secret object definition files to source code repositories.
Enabling Encryption at Rest for Secrets so they are stored encrypted in ETCD.
Also the way kubernetes handles secrets. Such as:

A secret is only sent to a node if a pod on that node requires it.
Kubelet stores the secret into a `tmpfs` so that the secret is not written to disk storage.
Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.
Read about the protections and risks of using secrets here

Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, HashiCorp Vault. I hope to make a lecture on these in the future.

#### to convert a secret to base64
```
echo -n 'string' | base64
```

## kubectl replace vs kubectl apply
`kubectl apply` uses the (declarative approach) provided spec to create a resource if it does not exist and update, i.e., patch, it if it does. The `spec` provided to apply need only contain the required parts of a `spec`, when creating a resource the API will use defaults for the rest and when updating a resource it will use its current values.
`apply` -> `apply`

The `kubectl replace` (imperative approach) completely replaces the existing resource with the one defined by the provided spec. replace wants a complete spec as input, including read-only properties supplied by the API like `.metadata.resourceVersion`, `.spec.nodeName` for pods, `.spec.clusterIP` for services, and `.secrets` for service accounts. kubectl has some internal tricks to help you get that right, but typically the use case for replace is getting a resource spec, changing a property, and then using that changed, complete spec to replace the existing resource.
`create` -> `replace`

The `kubectl replace` command has a `--force` option which actually does not use the replace, i.e., `PUT, API endpoint`. It forcibly deletes (`DELETE`) and then recreates, (`POST`) the resource using the provided spec.

## Assign Nodes for pods
Label Node and Assign Pods to Nodes
```
$ kubectl label nodes <node-name> <label-key>=<label-value>
```
Example:
```
kubectl label nodes k8snode01 disktype=ssd
```
You can then assign Pods to the Nodes.
```
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

## Tips
- applications = deployments
- to get node info check pods
Here's a quick tip. In the exam, you won't know if what you did is correct or not as in the practice tests in this course. You must verify your work yourself. For example, if the question is to create a pod with a specific image, you must run the the kubectl `describe` pod command to verify the pod is created with the correct name and correct image.

```
$ vim ~/.bashrc
# kubectl alias
alias k='kubectl'

# Create resources
alias kcf='kubectl create -f'
alias kaf='kubectl apply -f'

# List resources
alias kgp='kubectl get pods'
alias kgpa='kubectl get pods --all-namespaces'
alias kgd='kubectl get deployments'
alias kgs='kubectl get service'
alias kgh='kubectl get hpa'

# Delete resources
alias kd='kubectl delete'
alias kdp='kubectl delete pods'
alias kdd='kubectl delete deployments'
alias kgs='kubectl delete service'

$ source ~/.bashrc
```

### debug connectivity within service
kubectl exec -ti <<pod-name-1>> -- sh

  - now from within the pod
    ```
    nc -z -v <<pod-name-2>> <<port-number>>
    ```
    or
    ```
     nc -z -v -w 1 <<pod-name-2>> <<port-number>>
    ```

  - check network policies to check if it's only the default
    ```
    kubectl get netpol
    ```
  - if no create one and edit one
    ```
    kubectl get netpol default -o yaml > netpol-defn.yaml
    ```
## Taint/Toleration/Affinity
**Node affinity** is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.

**Tolerations** are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

**Taints** and **tolerations** work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

Concepts
You add a `taint` to a node using `kubectl taint`. For example,
```
kubectl taint nodes node1 key1=value1:NoSchedule
```

places a `taint` on node node1. The `taint` has key key1, value value1, and `taint` effect `NoSchedule`. This means that no pod will be able to schedule onto node1 unless it has a matching toleration.

To remove the `taint` added by the command above, you can run:
```
kubectl taint nodes node1 key1=value1:NoSchedule-
```

You specify a toleration for a pod in the PodSpec. Both of the following tolerations "match" the taint created by the kubectl `taint` line above, and thus a pod with either toleration would be able to schedule onto node1:
```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

## Admission Controllers
An admission controller is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but after the request is authenticated and authorized. The controllers consist of the list below, are compiled into the kube-apiserver binary, and may only be configured by the cluster administrator. In that list, there are two special controllers: MutatingAdmissionWebhook and ValidatingAdmissionWebhook. These execute the mutating and validating (respectively) admission control webhooks which are configured in the API.

Admission controllers may be "validating", "mutating", or both. Mutating controllers may modify related objects to the requests they admit; validating controllers may not.
Admission controllers limit requests to create, delete, modify objects or connect to proxy. They do not limit requests to read objects.

### Which plugins are enabled by default?
To see which admission plugins are enabled:
```
kube-apiserver -h | grep enable-admission-plugins
```

In the current version, the default ones are:
```
CertificateApproval, CertificateSigning, CertificateSubjectRestriction, DefaultIngressClass, DefaultStorageClass, DefaultTolerationSeconds, LimitRanger, MutatingAdmissionWebhook, NamespaceLifecycle, PersistentVolumeClaimResize, Priority, ResourceQuota, RuntimeClass, ServiceAccount, StorageObjectInUseProtection, TaintNodesByCondition, ValidatingAdmissionWebhook
```

### Important Errors
```
Error from server: error when creating "/root/pod-with-conflict.yaml": admission webhook "webhook-server.webhook-demo.svc" denied the request: runAsNonRoot specified, but runAsUser set to 0 (the root user)
```

### Why pv and pvc isn't enough
The StorageClass used by the PVC uses `WaitForFirstConsumer` volume binding mode. This means that the persistent volume will not bind to the claim until a pod makes use of the PVC to request storage.

Create a pod that has the volume mount described and the name of the pvc is important here

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx
    resources: {}
    volumeMounts:
    - mountPath: /var/www/html
      name: local-storage
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: local-storage
    persistentVolumeClaim:
      claimName: local-pvc
status: {}
```
### Change runtime config for a resource
#### take backup
```
cp -v /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.backup
```
#### change file
```
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
```
- command:
    - kube-apiserver
    - --advertise-address=10.18.17.8
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --runtime-config=rbac.authorization.k8s.io/v1alpha1 --> This one 
```
#### save the file and run this one to check if it was successful
```
kubectl get po -n kube-system
```
### Install a plugin (kubectl convert)
#### download the package
```
curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
```

#### change permissions to move it to the binaries
```
chmod +x kubectl-convert
mv kubectl-convert /usr/locl/bin/kubectl-convert
```

#### check if successful
```
kubectl --version
```

#### use it to change output version of an ingress file
```
kubectl-convert -f ingress-old.yaml --output-version networking.k8s.io/v1 > ingress-new.yaml
```

#### check if it was successful
```
kubectl apply -f ingress-new.yaml &
kubectl get ing ingress-space -o yaml | grep apiVersion
```

### rolling update vs recreate
#### recreate
```
strategy:
    type: Recreate
```

#### rollingUpdate
```
strategy:
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
  type: RollingUpdate
```

### CronJob vs Job
#### Job is
```
apiVersion: batch/v1
kind: Job
metadata:
  name: throw-dice-pod
spec:
  backoffLimit: 15
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: throw-dice
          image: kodekloud/throw-dice
      restartPolicy: Never
```

#### CronJob is job inside Cron schedule
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: throw-dice-cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 1
      parallelism: 1
      backoffLimit: 25 # This is so the job does not quit before it succeeds.
      activeDeadlineSeconds: 20
      template:
        spec:
          containers:
          - name: throw-dice
            image: kodekloud/throw-dice
          restartPolicy: Never
```

### Different kind of secrets
#### generic secret
like `env` variables
```
envFrom:
  - secretRef:
      name: db-secret
```

#### volume mount secret
like a volume
```
volumes:
  - name: default-token-mz49x
    secret:
      defaultMode: 420
      secretName: default-token-mz49x
```

## Useful resources:
1. https://blog.atomist.com/kubernetes-apply-replace-patch/
2. https://medium.com/payscale-tech/imperative-vs-declarative-a-kubernetes-tutorial-4be66c5d8914
3. https://komodor.com/learn/the-ultimate-kubectl-cheat-sheet/
4. https://computingforgeeks.com/kubectl-cheat-sheet-for-kubernetes-cka-exam-prep/