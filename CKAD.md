## YAML Manifests

All k8s definition files have four required sections:

- `apiVersion`
- `kind`
- `metadata`
- `spec` (or sometimes `data`)

## Environment Setup

Setup kubectl alias:

```shell
$ alias kc=kubectl
```

Configure Vim for YAML files:

```shell
$ echo 'autocmd Filetype yaml setlocal ts=2 sts=2 sw=2 expandtab autoindent' > ~/.vimrc
```

Open a tmux session with two panes:

```
$ tmux
Ctrl+b %
```

# Pods

## Creating Pods

### pod-definition.yml

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

## Editing Existing Pods

In any of the practical quizzes if you are asked to **edit an existing POD**, please note the following:

- If you are given a pod definition file, edit that file and use it to create a new pod.

- **If you are not given a pod definition file**, you may extract the definition to a file using the below command:
  
  ```shell
  kubectl get pod <pod-name> -o yaml > pod-definition.yaml
  ```
  
  Then edit the file to make the necessary changes, delete and re-create the pod.

- Use the `kubectl edit pod <pod-name>` command to edit pod properties.

Create just a pod via imperative command with label:

```shell
$ kc run --generator=run-pod/v1 nginx --image=nginx -l tier=frontend
```

Create pod manifest for nginx:

```shell
$ kc run nginx --image=nginx --dry-run -o yaml
```

Get pod info:

```shell
$ kc describe pod nginx-pod
```

Get more details of pods:

```shell
$ kc get pod -o wide
```

Update an existing pod from changed source yaml:

```shell
$ kc apply -f redis-pod.yaml
```

Get running pod defintion from k8s and edit/update:

```shell
$ kc get po <pod-name> -o yaml > pod-definition.yaml
$ vim pod-definition.yaml
$ kc apply -f pod-definition.yaml
```

Edit running pod definition directly:

```shell
$ kc edit po <pod-name>
```

Delete all pods in default namespace:

```shell
$ kc delete po --all
```

Shell into (if possible) a running pod:

```shell
$ kc exec -it pod-name /bin/sh (or /bin/bash, if available)
```

# ReplicaSet

A template section under spec can define a child resource, e.g., a pod resource under replicaSet's spec.template section. Do not include apiVersion or kind in the template.

A replicaset requires the selector section with matchLabels defining the pod label to replicate on. It is still required to include the pod template even if the pods already exist before creating the replicaset, as any scale-up of more or failed pods will be done from this template.

Create a replicaset:

```shell
$ kc create -f replicaset-definition.yaml
```

List all replicaset:

```shell
$ kc get replicaset
```

Delete a replicaset: <div style="text-align: right"> <span style="color:blue"> _*Deletes all underlying pods_ </span> </div>

```shell
$ kc delete replicaset nginx-replicaset 
```

Scale a replicaset to a different number of pods:

```shell
$ kc scale --replicas=X rs nginx-replicaset
```

Edit running replicatset definition directly:

```shell
$ kc edit rs <replicaset>
```

Update a replicaset with the replace command via definition file:

```shell
$ kc replace -f replicaset-definition.yaml
```

Get definition file from kubectl:

```shell
$ kc get rs ngnix-replicaset -o yaml > nginx-replicaset.yaml
```

# Deployment

Create deployment direct from kubectl:

```shell
$ kc run nginx --image=nginx
```

Create a deployment manifest with 4 replicas of nginx and save to a file:

```shell
$ kc run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml
```

Create a service with nodeport 30080 exposed for access:

```shell
$ kc create svc nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml
```

Create a service onto a deployment:

```shell
$ kc expose deployment webapp --type=NodePort --port=8080 --name=webapp-service --dry-run -o yaml > webapp-service.yaml
$ vim webapp-service.yaml  ports:    nodePort: 30082
$ kc create -f webapp-service.yaml
```

# Namespace

View all resources that exist in namespace of current context:

```shell
$ kc get all
```

Switch context to a different namespace, and verify

```shell
$ kc config set-context $(kubectl config current-context) --namespace=elastic-stack
$ kc config view | grep namespace
```

# ConfigMap

Create a configmap imperatively with literal values:

```shell
$ kc create cm cmname --from-literal=KEY=VALUE
```

Create a configmap imperatively with a key-value file:

```shell
$ kc create cm configmap-name --from-file=keyvalue.properties
```

Create a configmap declaratively from manifest file (uses a "data" header instead of "spec" in the file):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:  
  name: configmap-name
data:  
  KEY1: value1
  KEY2: value2
```

Inject a configmap into a pod defintion:

```yaml
spec:  
  envFrom:    
    - configMapRef:        
        name: configmap-name
```

# Secret

To create a secret file, encode values in base64:

```shell
$ echo -n 'password' | base64
```

To decode hashed values:

```shell
$ echo -n 'hAsH3dPAssW0rD' | base64 --decode
```

To attach a secret to a pod:

```yaml
spec:
  envFrom:
    - secretRef:
        name: secret-name
```

# securityContext

Set user at pod-level (if not set in container-level):

```yaml
spec:
  securityContext:
    runAsUser: 1001
```

Set user and capabilities at container level (this will override pod-level `securityContext`):

```yaml
spec:
  containers:
    - name: app-container
      securityContext:
        runAsUser: 1001
        capabilities:
          add:
            - "MAC_ADMIN"
```

# serviceAccount

*(Note: RBAC is also needed to apply appropriate permissions to a service account, but that will be covered later.)*

Get list of service accounts:

```shell
$ kc get sa
```

Create a service account:

```shell
$ kc create sa sa-name
```

Add a service account to a pod or deployment:

```yaml
spec:
  serviceAccount: sa-name
```

# Resource Requirements

### resources.requests

Setting resource minimum requirements:

- `cpu` in number of cores (default: 0.5, for half-core or half-hyperthread)
- `memory` in bytes (default: 256Mi, for 256 megabytes)

```yaml
spec:
  containers:
  - name: app-container
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
```

### resources.limits

Setting resource limits:

- `cpu` defaults to 1 core, throttled when trying to exceed
- `memory` defaults to 512Mi, terminated when exceeding

```yaml
spec:
  containers:
  - name: app-container
    resources:
      limits:
        memory: "2Gi"
        cpu: 2
```

# Taints and Tolerations

Taint nodes to prevent intolerant pods from being scheduled on those nodes.

- `taint-effect` can be one of:
  - NoSchedule - applies to new pods being scheduled
  - PreferNoSchedule
  - NoExecute - will terminate and reschedule any pods on a newly tainted node that don't have the toleration
- Tolerations on pods do not guarantee that they are scheduled only on tainted nodes (NodeAffinity does this)

```shell
$ kc taint no node-name instanceType=Compute:PreferNoSchedule
```

Add tolerations to pods for being acceptable to schedule on tainted nodes:

```yaml
spec:
  tolerations:
  - key: "instanceType"
    operator: "Equal"
    value: "Compute"
    effect: "PreferNoSchedule"
```

Remove a taint from a node by key (and effect):

```shell
$ kc taint no node-name instanceType:PreferNoSchedule-
```

# nodeSelector

Label nodes for node selectors:

```shell
$ kc label no node-name memSize=Large
```

Set which nodes a pod would be nice to be scheduled on with `nodeSelector`:

```yaml
spec:
  nodeSelector:
    memSize: Large
```

# nodeAffinity

Use `nodeAffinity` to get specific on pod-to-node scheduling preferences:

- `nodeAffinity` types available/planned:
  - `requiredDuringSchedulingIgnoredDuringExecution`
  - `preferredDuringSchedulingIgnoredDuringExecution`
  - `requiredDuringSchedulingRequiredDuringExecution`
- `operator`s available:
  - `In`
  - `NotIn`
  - `Exists`
  - (others in documentation)

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: memSize
            operator: In
            values:
            - Large
          - key: memSize
            operator: NotIn
            values: 
            - Small
```

# Multi-Container Pods

Three types of multi-container pod patterns that take advantage of shared context (`localhost` on the network to one another, automatically shared `volumes`):

- *sidecar*: executes auxilliary operations to the main app container (e.g., logging agent)
- *adapter*: executes local context-specific operations (e.g., logging parser/transformer to standard format, pre-sidecar)
- *ambassador*: provides local proxy to environment-specific resources (e.g., remove need for configuration of environment-specific database URL)

# Readiness Probes

Test for true container readiness to receive traffic and signal to Kubernetes when a container reaches this condition.

Each container in a pod can be configured with its own readiness probe, and the pod is not in a ready state until all containers are ready. Different types of probes are available for configuration:

- hitting an HTTP endpoint (`httpGet`)
- checking for a listening TCP port (`tcpSocket`)
- executing a custom script that exits 0 on success (`exec: command: - command - argument`)

```yaml
spec:
  containers:
  - name: app
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 8
```

# Liveness Probes

On a running container, maintain status of a validly running application by configuring liveness probes (`livenessProbe`) to regularly check a container's status. Configuration options are the same as `readinessProbe`.

# Container Logging

View live running pod logs:

```shell
$ kc logs -f pod-name
```

In a multi-container pod, logs can only be shown from one container, and must be specified:

```shell
$ kc logs -f pod-name container-name
```

# Cluster Monitoring

Open-source cluster monitoring solutions such as Prometheus and Elastic stack are covered in the CKA exam.

CKAD only requires awareness of [Metrics Server](https://github.com/kubernetes-incubator/metrics-server), an in-memory-only slim replacement for Heapster that provides node- and pod-level metrics.

Metrics Server reads node and pod metrics by querying the `kubelet` API on each node, which in turn uses `cAdvisor` to provide pod/container metrics for resources each node is running.

To enable metrics-server on Minikube:

```shell
$ minikube addons enable metrics-server
```

To deploy metrics-server to any other cluster:

```shell
$ git clone https://github.com/kubernetes-incubator/metrics-server.git
$ cd metrics-server
$ kc create -f deploy/1.8+/ # create command can take a directory of YAML files as an argument
```

Inspect the resource usage of each node:

```shell
$ kc top node
```

Inspect the resource usage of each pod:

```shell
$ kc top pod
```
