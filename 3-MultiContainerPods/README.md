- [**Pods and Containers**](#pods-and-containers)
- [**Multi Container Pods**](#multi-container-pods)
  - [**Sidecar Pattern**](#sidecar-pattern)
  - [**Adapter Pattern**](#adapter-pattern)
  - [**Ambassador Pattern**](#ambassador-pattern)
- [**References**](#references)
# **Pods and Containers**

In Kubernetes, Pods are the single deployable units. If an application is to be deployed it has to be deployed in a Pod as a container. We can declare more than one container in a Pod specification.

Pod is an extra layer of abstraction around a container so that you can have different types of containers—Docker, rkt, etc—managed by a single orchestration system. It also allows pod-level control of restart policies, networking, and filesystem sharing.

# **Multi Container Pods**

Multi Container pods are generally required,

- When the containers _`lifecycles are identical`_, or when they must run on the same node. The most common scenario is that you need to locate and manage a helper process on the same node as the primary container.

- Another reason to group containers into a single pod is to make communication between them easier. Inter-process communication and shared volumes (writing to a shared file or directory) are two ways for these containers to communicate (semaphores or shared memory).

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
      image: myapp
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
      ports:
        - containerPort: 8080
    - name: log-agent
      image: log-agent
      resources:
        limits:
          memory: "128Mi"
          cpu: "0.5"
```

The `sidecar pattern`, the `adapter pattern`, and the `ambassador pattern` are three common design patterns and use-cases for combining multiple containers into a single pod.

![Multi Container Pods Patterns](https://matthewpalmer.net/kubernetes-app-developer/multi-container-pod-design.png)

## **Sidecar Pattern**

The sidecar pattern consists of a main application and a helper container with a responsibility that is important main application but isn't necessarily part of it.

Logging utilities, sync services, watchers, and monitoring agents are the most common sidecar containers. Because it's pointless to have a logging container running while the main application isn't, we create a pod that contains both the main application and the sidecar container. Another advantage of moving the logging work is that if the logging code is bad, the problem will be isolated to that container an exception thrown in the non-essential logging code won't bring the main application down.

## **Adapter Pattern**

The adapter pattern is used to standardise and normalise application output or monitoring data for aggregation.

As an example, consider a cluster-level monitoring agent that monitors response times. In our cluster, we have a Ruby application that writes request timing in the format [HOST] - [DURATION], while another Node.js application writes it in [HOST] - [START DATE].

The monitoring agent can only accept data in the following format: [HOST] - [DATE] - [DURATION]. It will be difficult for the developer to change the format, and it may have an impact on other dependencies. The better option is to include adapter containers that convert the output to the desired format. The application developer can then simply update the pod definition to include the adapter container and enjoy free monitoring.

## **Ambassador Pattern**

The ambassador pattern can be used to connect containers to the outside world. An ambassador container is essentially a proxy that allows other containers to connect to a port on localhost, while the ambassador container can redirect these connections to different environments based on the needs of the cluster.

One of the best applications for the ambassador pattern is providing database access. When developing locally, you will most likely want to use your local database, whereas your test and production deployments will require different databases.

Because the URI changes in each environment, it is best for the application to always connect to localhost and delegate the responsibility of connecting to the correct database to an ambassador container.

# **References**

1. [Multi-Container Pod Design Patterns in Kubernetes](https://matthewpalmer.net/kubernetes-app-developer/articles/multi-container-pod-design-patterns.html)