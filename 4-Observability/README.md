- [**Pod Lifecycle**](#pod-lifecycle)
  - [**Pod lifetime**](#pod-lifetime)
  - [**Pod phase**](#pod-phase)
  - [**Container states**](#container-states)
  - [**Container restart policy**](#container-restart-policy)
  - [**Pod Conditions**](#pod-conditions)
- [**Debug in Kubernetes**](#debug-in-kubernetes)
  - [**Debugging with an ephemeral debug container**](#debugging-with-an-ephemeral-debug-container)
  - [**Debugging using a copy of the Pod**](#debugging-using-a-copy-of-the-pod)
    - [**Copying a Pod while adding a new container**](#copying-a-pod-while-adding-a-new-container)
  - [**Copying a Pod while changing its command**](#copying-a-pod-while-changing-its-command)
  - [**Copying a Pod while changing container images**](#copying-a-pod-while-changing-container-images)
- [**Probes**](#probes)
  - [**Configure Probes**](#configure-probes)
  - [**Check mechanisms**](#check-mechanisms)
    - [**`exec` Probes**](#exec-probes)
    - [**`grpc` Probes**](#grpc-probes)
    - [**`httpGet` Probe**](#httpget-probe)
    - [**`tcpSocket` Probes**](#tcpsocket-probes)
  - [**Types of Probe**](#types-of-probe)
    - [**`livenessProbe`**](#livenessprobe)
    - [**`startupProbe`**](#startupprobe)
    - [**`readinessProbe`**](#readinessprobe)
- [**Logging in K8**](#logging-in-k8)
- [**Monitoring in K8**](#monitoring-in-k8)
  - [**Enabling Metrics Server**](#enabling-metrics-server)
  - [**Check Metrics**](#check-metrics)
- [**References**](#references)

# **Pod Lifecycle**

Pods follow a defined lifecycle, starting in the `Pending` [phase](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase), moving through `Running` if at least one of its primary containers starts OK, and then through either the `Succeeded` or `Failed` phases depending on whether any container in the Pod terminated in failure.

Whilst a Pod is running, the kubelet is able to restart containers to handle some kind of faults. Within a Pod, Kubernetes tracks different container [states](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-states) and determines what action to take to make the Pod healthy again.

In the Kubernetes API, Pods have both a specification and an actual status. The status for a Pod object consists of a set of [Pod conditions](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions). You can also inject [custom readiness information](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate) into the condition data for a Pod, if that is useful to your application.

Pods are only [scheduled](https://kubernetes.io/docs/concepts/scheduling-eviction/) once in their lifetime. Once a Pod is scheduled (assigned) to a Node, the Pod runs on that Node until it stops or is [terminated](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination).

## **Pod lifetime**

Like individual application containers, Pods are considered to be relatively ephemeral (rather than durable) entities. Pods are created, assigned a unique ID ([UID](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#uids)), and scheduled to nodes where they remain until termination (according to restart policy) or deletion. If a [Node](https://kubernetes.io/docs/concepts/architecture/nodes/) dies, the Pods scheduled to that node are [scheduled for deletion](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-garbage-collection) after a timeout period.

Pods do not, by themselves, self-heal. If a Pod is scheduled to a [node](https://kubernetes.io/docs/concepts/architecture/nodes/) that then fails, the Pod is deleted; likewise, a Pod won't survive an eviction due to a lack of resources or Node maintenance. Kubernetes uses a higher-level abstraction, called a [controller](https://kubernetes.io/docs/concepts/architecture/controller/), that handles the work of managing the relatively disposable Pod instances.

A given Pod (as defined by a UID) is never "rescheduled" to a different node; instead, that Pod can be replaced by a new, near-identical Pod, with even the same name if desired, but with a different UID.

When something is said to have the same lifetime as a Pod, such as a [volume](https://kubernetes.io/docs/concepts/storage/volumes/), that means that the thing exists as long as that specific Pod (with that exact UID) exists. If that Pod is deleted for any reason, and even if an identical replacement is created, the related thing (a volume, in this example) is also destroyed and created anew.

![https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)

****Pod diagram****

*A multi-container Pod that contains a file puller and a web server that uses a persistent volume for shared storage between the containers.*

## **Pod phase**

A Pod's `status` field is a [PodStatus](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.24/#podstatus-v1-core) object, which has a `phase` field.

The phase of a Pod is a simple, high-level summary of where the Pod is in its lifecycle. The phase is not intended to be a comprehensive rollup of observations of container or Pod state, nor is it intended to be a comprehensive state machine.

The number and meanings of Pod phase values are tightly guarded. Other than what is documented here, nothing should be assumed about Pods that have a given `phase` value.

- A POD has a _*pod status*_ and _*pod conditions*_. The POD status tells us where the POD is in its lifecycle.
  
- When a POD is first created it is in a `Pending` state. This is when the scheduler tries to figure out where to place the POD.
  
  - If the scheduler cannot find the node to place the POD, it remains in a `Pending` state. 
    - To find out why it's stuck in a `Pending` state, run the kubectl describe pod command and it will tell you exactly why.
  
- Once the POD scheduled, it goes into a `ContainerCreating` status where the images required for the application are pulled and the container starts.
  
- Once _all the containers_ in a POD starts it goes into a `Running` state where it continues to be `Running` until the program completes successfully or is terminated.

![Pod LifeCycle](https://sysdig.com/wp-content/uploads/Understanding-Pod-Pending-problems-02.png)

Here are the possible values for `phase`:

| Value	| Description |
| :---: | :---: |
| Pending |	The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run. This includes time a Pod spends waiting to be scheduled as well as the time spent downloading container images over the network. |
| Running |	The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting. |
| Succeeded | All containers in the Pod have terminated in success, and will not be restarted. |
| Failed | All containers in the Pod have terminated, and at least one container has terminated in failure. That is, the container either exited with non-zero status or was terminated by the system. |
| Unknown |	For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running. |

 > **Note: When a Pod is being deleted, it is shown as `Terminating` by some kubectl commands. This `Terminating` status is not one of the Pod phases. A Pod is granted a term to terminate gracefully, which defaults to 30 seconds. You can use the flag `--force` to [terminate a Pod by force](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination-forced).**

If a node dies or is disconnected from the rest of the cluster, Kubernetes applies a policy for setting the `phase` of all Pods on the lost node to Failed.

## **Container states**

As well as the [phase](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase) of the Pod overall, Kubernetes tracks the state of each container inside a Pod.

Once the [scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) assigns a Pod to a Node, the kubelet starts creating containers for that Pod using a [container runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes). There are three possible container states: `Waiting`, `Running`, and `Terminated`.

To check the state of a Pod's containers, you can use `kubectl describe pod <name-of-pod>`. The output shows the state for each container within that Pod.

Each state has a specific meaning:

| State	| Description |
| :---: | :---: |
| **`Waiting`** | If a container is not in either the `Running` or `Terminated` state, it is `Waiting`. A container in the `Waiting` state is still running the operations it requires in order to complete start up: for example, pulling the container image from a container image registry, or applying [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) data. When you use `kubectl` to query a Pod with a container that is `Waiting`, you also see a Reason field to summarize why the container is in that state. |

| **`Running`** | The `Running` status indicates that a container is executing without issues. When you use `kubectl` to query a Pod with a container that is `Running`, you also see information about when the container entered the `Running` state. |

| **`Terminated`** | A container in the `Terminated` state began execution and then either ran to completion or failed for some reason. When you use `kubectl` to query a Pod with a container that is `Terminated`, you see a reason, an exit code, and the start and finish time for that container's period of execution. |

You can use [container lifecycle hooks](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/) to trigger events to run at certain points in a container's lifecycle.
- If there was a `postStart` hook configured, it has already executed and finished.
- If a container has a `preStop` hook configured, this hook runs before the container enters the `Terminated` state.

## **Container restart policy**

The `spec` of a Pod has a `restartPolicy` field with possible values:
- Always
- OnFailure
- Never
 
The default value is Always.

The `restartPolicy` applies to all containers in the Pod. `restartPolicy` only refers to restarts of the containers by the kubelet on the same node. After containers in a Pod exit, the kubelet restarts them with an exponential back-off delay (10s, 20s, 40s, …), that is capped at five minutes. Once a container has executed for 10 minutes without any problems, the kubelet resets the restart backoff timer for that container.

## **Pod Conditions**


A Pod has a PodStatus, which has an array of PodConditions through which the Pod has or has not passed:

- `PodScheduled`: the Pod has been scheduled to a node.
- `ContainersReady`: all containers in the Pod are ready.
- `Initialized`: all init containers have completed successfully.
- `Ready`: the Pod is able to serve requests and should be added to the load balancing pools of all matching Services.

| Field name | Description |
| :---: | :---: |
| type |	Name of this Pod condition. |
| status |	Indicates whether that condition is applicable, with possible values "True", "False", or "Unknown". |
| lastProbeTime |	Timestamp of when the Pod condition was last probed. |
| lastTransitionTime |	Timestamp for when the Pod last transitioned from one status to another. |
| reason |	Machine-readable, UpperCamelCase text indicating the reason for the condition's last transition. |
| message |	Human-readable message indicating details about the last status transition. |


# **Debug in Kubernetes**

If the container image includes debugging utilities:

```bash
kubectl exec ${POD_NAME} -c ${CONTAINER_NAME} -- ${CMD} ${ARG1} ${ARG2} ... ${ARGN}
```
## **Debugging with an ephemeral debug container**
`kubectl exec` is insufficient because a container has crashed or a container image doesn't include debugging utilities

```bash
kubectl debug -it ${TARGET_POD_NAME} --image=${NEW_IMAGE} --target=${TARGET_CONTAINER_NAMESPACE}
```

This command adds a new busybox container and attaches to it. The `--target` parameter targets the process namespace of another container. It's necessary here because `kubectl run` does not enable process namespace sharing in the pod it creates.

## **Debugging using a copy of the Pod**

We can't run `kubectl exec` to troubleshoot your container if your container image does not include a shell or if your application crashes on startup. In these situations you can use `kubectl debug` to create a copy of the Pod with configuration values changed to aid debugging.

### **Copying a Pod while adding a new container**

Run this command to create a copy of TARGET_POD named COPY_POD that adds a new NEW_CONTAINER_IMAGE (Ubuntu) container for debugging:

```bash
kubectl debug ${TARGET_POD_NAME} -it --image=${NEW_CONTAINER_IMAGE} --share-processes --copy-to=${COPY_POD_NAME}
```

> **Note**:
> 
> - `kubectl debug` automatically generates a container name if you don't choose one using the `--container` flag.
> - The `-i` flag causes `kubectl debug` to attach to the new container by default. You can prevent this by specifying `--attach=false`. If your session becomes disconnected you can reattach using `kubectl attach`.
> - The `--share-processes` allows the containers in this Pod to see processes from the other containers in the Pod.

## **Copying a Pod while changing its command**

Run kubectl debug to create a copy of this Pod with the command changed to an interactive shell:

```bash
kubectl debug ${TARGET_POD_NAME} -it --copy-to=${COPY_POD_NAME} --container=${TARGET_CONTAINER_NAME} -- sh #${NEW_COMMAND}
```

> **Note**:
> To change the command of a specific container you must specify its name using `--container` or `kubectl debug` will instead create a new container to run the command you specified.
> The `-i` flag causes `kubectl debug` to attach to the container by default. You can prevent this by specifying `--attach=false`. If your session becomes disconnected you can reattach using `kubectl attach`.

## **Copying a Pod while changing container images**

Use kubectl debug to make a copy and change its container image to ubuntu:

```bash
kubectl debug ${TARGET_POD_NAME} --copy-to=${COPY_POD_NAME} --set-image=*=ubuntu
```
The syntax of `--set-image` uses the same container_name=image syntax as kubectl set image. `*=ubuntu` means change the image of all containers to ubuntu.

# **Probes**

A probe is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the `kubelet` either executes code within the container, or makes a network request.

## **Configure Probes**

Probes have a number of fields that you can use to more precisely control the behavior of liveness and readiness checks:

- `initialDelaySeconds`: Number of seconds after the container has started before liveness or readiness probes are initiated. Defaults to 0 seconds. Minimum value is 0.
- `periodSeconds`: How often (in seconds) to perform the probe. Default to 10 seconds. Minimum value is 1.
- `timeoutSeconds`: Number of seconds after which the probe times out. Defaults to 1 second. Minimum value is 1.
- `successThreshold`: Minimum consecutive successes for the probe to be considered successful after having failed. Defaults to 1. Must be 1 for liveness and startup Probes. Minimum value is 1.
- `failureThreshold`: When a probe fails, Kubernetes will try `failureThreshold` times before giving up. Giving up in case of liveness probe means restarting the container. In case of readiness probe the Pod will be marked Unready. Defaults to 3. Minimum value is 1.

## **Check mechanisms**
There are four different ways to check a container using a probe. Each probe must define exactly one of these four mechanisms:
### **`exec` Probes**

Executes a specified command inside the container. The diagnostic is considered successful if the command exits with a status code of 0.

| Field	| Description |
| - | - |
| `command` (string array) | Command is the command line to execute inside the container, the working directory for the command is root ('/') in the container's filesystem. The command is simply exec'd, it is not run inside a shell, so traditional shell instructions ('\|', etc) won't work. To use a shell, you need to explicitly call out to that shell. Exit status of 0 is treated as live/healthy and non-zero is unhealthy. |

### **`grpc` Probes**

Performs a remote procedure call using gRPC. The target should implement gRPC health checks. The diagnostic is considered successful if the `status` of the response is `SERVING`.
gRPC probes are an alpha feature and are only available if you enable the `GRPCContainerProbe` feature gate.

| Field	| Description |
| - | - |
| `service` (string) | Service is the name of the service to place in the gRPC [HealthCheckRequest](https://github.com/grpc/grpc/blob/master/doc/health-checking.md). If this is not specified, the default behavior is defined by gRPC. |
| `port` (integer) | Port number of the gRPC service. Number must be in the range 1 to 65535. |


### **`httpGet` Probe**

Performs an HTTP GET request against the Pod's IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.

| Field	| Description |
| - | - |
| `host` (string) | Host name to connect to, defaults to the pod IP. You probably want to set "Host" in httpHeaders instead. |
| `httpHeaders` (HTTPHeader array) | Custom headers to set in the request. HTTP allows repeated headers. |
| `path` (string) | Path to access on the HTTP server. |
| `port` | Name or number of the port to access on the container. Number must be in the range 1 to 65535. Name must be an IANA_SVC_NAME. |
| `scheme` (string) | Scheme to use for connecting to the host. Defaults to HTTP. |

### **`tcpSocket` Probes**

Performs a TCP check against the Pod's IP address on a specified port. The diagnostic is considered successful if the port is open. If the remote system (the container) closes the connection immediately after it opens, this counts as healthy.

| Field	| Description |
| - | - |
| `host` (string) | Optional: Host name to connect to, defaults to the pod IP. |
| `port` | Name or number of the port to access on the container. Number must be in the range 1 to 65535. Name must be an IANA_SVC_NAME. |

	
## **Types of Probe**

### **`livenessProbe`**

Indicates whether the container is running. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a liveness probe, the default state is `Success`.

Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
      ports:
        - containerPort: 8080
      # HTTP Liveness Probe
      livenessProbe:
        httpGet:
          path: /api/ready
          port: 8080
          httpHeaders:
          - name: Custom-Header
            value: Awesome
        initialDelaySeconds: 10 # Delay before starting Liveness probe
        periodSeconds: 5 # How often to probe?
        failureThreshold: 8 # Consecutive failures to be considered failed

      # TCP Liveness Probe
      # livenessProbe:
      #   tcpSocket:
      #     port: 3306

      # Exec command
      # livenessProbe:
      #   exec:
      #     command:
      #       - cat
      #       - /app/is_ready

      # livenessProbe:
        # grpc:
        #   port: 2379

```

### **`startupProbe`**

Indicates whether the application within the container is started. _All other probes are disabled if a startup probe is provided, until it succeeds_. If the startup probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a startup probe, the default state is `Success`.

> **Note**: Protect slow starting containers with startup probes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
      ports:
        - containerPort: 8080
      # HTTP startup Probe
      startupProbe:
        httpGet:
          path: /healthz
          port: liveness-port
  
          httpHeaders:
          - name: Custom-Header
            value: Awesome
        initialDelaySeconds: 10 # Delay before starting Liveness probe
        periodSeconds: 10 # How often to probe?
        failureThreshold: 30 # Consecutive failures to be considered failed

      # TCP startup Probe
      # startupProbe:
      #   tcpSocket:
      #     port: 3306

      # Exec command
      # startupProbe:
      #   exec:
      #     command:
      #       - cat
      #       - /app/is_ready

      # startupProbe:
        # grpc:
        #   port: 2379

```

### **`readinessProbe`**

Indicates whether the container is ready to respond to requests. If the readiness probe fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is `Failure`. If a container does not provide a readiness probe, the default state is `Success`.

Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup, or depend on external services after startup. In such cases, you don't want to kill the application, but you don't want to send it requests either. Kubernetes provides readiness probes to detect and mitigate these situations. A pod with containers reporting that they are not ready does not receive traffic through Kubernetes Services.

> 📝 **Note:** 
> 
> Readiness probes runs on the container during its whole lifecycle.

> ⚠️ **Caution:** 
> 
> Liveness probes do not wait for readiness probes to succeed. If you want to wait before executing a liveness probe you should use `initialDelaySeconds` or a `startupProbe`.
> 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
      ports:
        - containerPort: 8080
      # HTTP Readiness Probe
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
          httpHeaders:
          - name: Custom-Header
            value: Awesome
        initialDelaySeconds: 10 # Delay before starting readiness probe
        periodSeconds: 5 # How often to probe?
        failureThreshold: 8 # Consecutive failures to be considered failed

      # TCP Readiness Probe
      # readinessProbe:
      #   tcpSocket:
      #     port: 3306

      # Exec command
      # readinessProbe:
      #   exec:
      #     command:
      #       - cat
      #       - /app/is_ready

```

# **Logging in K8**

To check the logs run following commands:

*Single Container Pods*:
```bash
kubectl logs <pod-name>
```
To see logs of previous instance (Say Before Restart)
```bash
kubectl logs <pod-name> -p
kubectl logs <pod-name> --previous
```
*Multi Container Pods*:
```bash
kubectl logs <pod-name> <container-name>
```
To see live logs add `-f` flag to command.
```bash
kubectl logs -f <pod-name> <container-name>
```

# **Monitoring in K8**

Kubernetes does not come with a full feature built in monitoring solution.

There are a number of open source solutions available today such as Metric Server, Prometheus, Elastic Stack and proprietary solutions like Datadog and Dynatrace.

Heapster was one of the original projects that enabled monitoring and analysis features for Kubernetes. We see a lot of reference online when you look for reference architectures on monitoring Kubernetes. However, Heapster is now deprecated
and a slimmed down version was formed known as the Metric Server.

We can have one metric server per Kubernetes cluster. The Metrics Server retrieves metrics from each of the Kubernetes nodes and pods, aggregates them and store them in memory.

> **Note:** metric server is only an in-memory monitoring solution and does not store the metrics on the disk.

## **Enabling Metrics Server**

If using Minikube
```bash
minikube addons enable metrics-server
```

Else 
```bash
git clone 
```

## **Check Metrics**

```bash
kubectl top node/pod
```

```bash
watch "kubectl top node/pod"
```

# **References**

- [Understanding the pod's status - Wang Wei](https://wangwei1237.github.io/Kubernetes-in-Action-Second-Edition/docs/Understanding_the_pods_status.html)

- [Understanding Kubernetes pod pending problems - Carlos Arilla](https://sysdig.com/blog/kubernetes-pod-pending-problems/)


