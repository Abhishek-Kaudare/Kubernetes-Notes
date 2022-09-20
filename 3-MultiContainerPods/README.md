- [**Pods and Containers**](#pods-and-containers)
- [**Multi Container Pods**](#multi-container-pods)
  - [**Sidecar Pattern**](#sidecar-pattern)
  - [**Adapter Pattern**](#adapter-pattern)
  - [**Ambassador Pattern**](#ambassador-pattern)
- [**References**](#references)
# **Pods and Containers**

In Kubernetes, Pods are the _single deployable units_. If an application is to be deployed it has to be deployed in a Pod as a container. We can declare more than one container in a Pod specification.

Pod is an _extra layer of abstraction around a container_ so that you can have different types of containers — Docker, rkt, etc—managed by a single orchestration system. It also allows pod-level control of restart policies, networking, and filesystem sharing.

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

The most common sidecar containers are:
- Logging utilities
- Sync services
- Watchersand 
- Monitoring agents 
 
Because it's pointless to have a logging container running while the main application isn't, we create a pod that contains both the main application and the sidecar container. Another advantage of moving the logging work is that if the logging code is bad, the problem will be isolated to that container an exception thrown in the non-essential logging code won't bring the main application down.

```yaml
# Example YAML configuration for the sidecar pattern.

# It defines a main application container which writes
# the current date to a log file every five seconds.

# The sidecar container is nginx serving that log file.
# (In practice, your sidecar is likely to be a log collection
# container that uploads to external storage.)

# To run:
#   kubectl apply -f pod.yaml

# Once the pod is running:
#   
#   (Connect to the sidecar pod)
#   kubectl exec pod-with-sidecar -c sidecar-container -it bash
#   
#   (Install curl on the sidecar)
#   apt-get update && apt-get install curl
#   
#   (Access the log file via the sidecar)
#   curl 'http://localhost:80/app.txt'

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-sidecar
spec:
  # Create a volume called 'shared-logs' that the
  # app and sidecar share.
  volumes:
  - name: shared-logs 
    emptyDir: {}

  # In the sidecar pattern, there is a main application
  # container and a sidecar container.
  containers:

  # Main application container
  - name: app-container
    # Simple application: write the current date
    # to the log file every five seconds
    image: alpine # alpine is a simple Linux OS image
    command: ["/bin/sh"]
    args: ["-c", "while true; do date >> /var/log/app.txt; sleep 5;done"]

    # Mount the pod's shared log file into the app 
    # container. The app writes logs here.
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  # Sidecar container
  - name: sidecar-container
    # Simple sidecar: display log files using nginx.
    # In reality, this sidecar would be a custom image
    # that uploads logs to a third-party or storage service.
    image: nginx:1.7.9
    ports:
      - containerPort: 80

    # Mount the pod's shared log file into the sidecar
    # container. In this case, nginx will serve the files
    # in this directory.
    volumeMounts:
    - name: shared-logs
      mountPath: /usr/share/nginx/html # nginx-specific mount path
```
## **Adapter Pattern**

The adapter pattern is used to standardise and normalise application output or monitoring data for aggregation.

As an example, consider a cluster-level monitoring agent that monitors response times. In our cluster, we have a Ruby application that writes request timing in the format [HOST] - [DURATION], while another Node.js application writes it in [HOST] - [START DATE].

The monitoring agent can only accept data in the following format: [HOST] - [DATE] - [DURATION]. It will be difficult for the developer to change the format, and it may have an impact on other dependencies. The better option is to include adapter containers that convert the output to the desired format. The application developer can then simply update the pod definition to include the adapter container and enjoy free monitoring.

```yaml
# Example YAML configuration for the adapter pattern.

# It defines a main application container which writes
# the current date and system usage information to a log file 
# every five seconds.

# The adapter container reads what the application has written and
# reformats it into a structure that a hypothetical monitoring 
# service requires. 

# To run:
#   kubectl apply -f pod.yaml

# Once the pod is running:
#   
#   (Connect to the application pod)
#   kubectl exec pod-with-adapter -c app-container -it sh
# 
#   (Take a look at what the application is writing.)
#   cat /var/log/top.txt
#   
#   (Take a look at what the adapter has reformatted it to.)
#   cat /var/log/status.txt

apiVersion: v1
kind: Pod
metadata:
  name: pod-with-adapter
spec:
  # Create a volume called 'shared-logs' that the
  # app and adapter share.
  volumes:
  - name: shared-logs 
    emptyDir: {}

  containers:

  # Main application container
  - name: app-container
    # This application writes system usage information (`top`) to a status 
    # file every five seconds.
    image: alpine
    command: ["/bin/sh"]
    args: ["-c", "while true; do date > /var/log/top.txt && top -n 1 -b >> /var/log/top.txt; sleep 5;done"]

    # Mount the pod's shared log file into the app 
    # container. The app writes logs here.
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log

  # Adapter container
  - name: adapter-container
    # This sidecar container takes the output format of the application
    # (the current date and system usage information), simplifies
    # and reformats it for the monitoring service to come and collect.
    
    # In this example, our monitoring service requires status files
    # to have the date, then memory usage, then CPU percentage each 
    # on a new line.
    
    # Our adapter container will inspect the contents of the app's top file,
    # reformat it, and write the correctly formatted output to the status file.
    image: alpine
    command: ["/bin/sh"]

    # A long command doing a simple thing: read the `top.txt` file that the
    # application wrote to and adapt it to fit the status file format.
    # Get the date from the first line, write to `status.txt` output file.
    # Get the first memory usage number, write to `status.txt`.
    # Get the first CPU usage percentage, write to `status.txt`.

    args: ["-c", "while true; do (cat /var/log/top.txt | head -1 > /var/log/status.txt) && (cat /var/log/top.txt | head -2 | tail -1 | grep
 -o -E '\\d+\\w' | head -1 >> /var/log/status.txt) && (cat /var/log/top.txt | head -3 | tail -1 | grep
-o -E '\\d+%' | head -1 >> /var/log/status.txt); sleep 5; done"]


    # Mount the pod's shared log file into the adapter
    # container.
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
```

## **Ambassador Pattern**

The ambassador pattern can be used to connect containers to the outside world. An _ambassador container is essentially a proxy_ that allows other containers to connect to a port on localhost, while the ambassador container can redirect these connections to different environments based on the needs of the cluster.

One of the best applications for the ambassador pattern is providing database access. When developing locally, you will most likely want to use your local database, whereas your test and production deployments will require different databases.

Because the URI changes in each environment, it is best for the application to always connect to localhost and delegate the responsibility of connecting to the correct database to an ambassador container.

# **References**

1. [Multi-Container Pod Design Patterns in Kubernetes](https://matthewpalmer.net/kubernetes-app-developer/articles/multi-container-pod-design-patterns.html)