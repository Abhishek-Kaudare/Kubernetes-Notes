- [**Labels and Selectors**](#labels-and-selectors)
- [**Annotations**](#annotations)
  - [**Attaching metadata to objects**](#attaching-metadata-to-objects)
  - [**Imperitive Commands**](#imperitive-commands)
- [**Deployments: Rollout & Rollback**](#deployments-rollout--rollback)
  - [**Rollout**](#rollout)
    - [**Deployment Strategy**](#deployment-strategy)
    - [**Creating a deployment**](#creating-a-deployment)
    - [**Get rollout status**](#get-rollout-status)
    - [**Get rollout history**](#get-rollout-history)
  - [**Rollback**](#rollback)
- [**Job**](#job)
  - [**Running a Job**](#running-a-job)
  - [**Writing a Job spec**](#writing-a-job-spec)
    - [**Pod Template**](#pod-template)
    - [**Pod Selector**](#pod-selector)
    - [**Parallel execution for Jobs**](#parallel-execution-for-jobs)
    - [**Controlling parallelism**](#controlling-parallelism)
    - [**Completion mode**](#completion-mode)
  - [**Handling Pod and container failures**](#handling-pod-and-container-failures)
    - [**Pod backoff failure policy**](#pod-backoff-failure-policy)
  - [**Job termination and cleanup**](#job-termination-and-cleanup)
  - [**Clean up finished jobs automatically**](#clean-up-finished-jobs-automatically)
    - [**TTL mechanism for finished Jobs**](#ttl-mechanism-for-finished-jobs)
  - [**Job patterns**](#job-patterns)
- [**CronJob**](#cronjob)
  - [**Cron schedule syntax**](#cron-schedule-syntax)
- [**Refernces**](#refernces)

# **Labels and Selectors**

1. Labels definition in yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
  # Labels for the object 
  labels:
    key1=value1
    key2=value2
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
Imperitive command
```bash
kubectl run pod nginx --image=nginx -l="key1=val2,key2=val2"
```
2. Get labels for all objects
```bash
kubectl get all --show-labels
```

3. Use Selectors

```bash
kubectl get all --selector key1=value1
```
OR
```bash
kubectl get all -l key1=value1
```
Match by key
```bash
kubectl get all -L key
```

Multiple Match
```bash
kubectl get all -l 'key in (val1,val2)'
```

Change the label for one of the pod to env=uat and list all the pods to verify

```bash
kubectl label pod/nginx-dev3 env=uat --overwrite
kubectl get pods --show-labels
```

Remove label with key 'env' and pods named nginx1, nginx2 and nginx3
```bash
kubectl label pod nginx(1..3} env-
``` 

Change value of label with key-value 'env=prod' and pods named nginx1, nginx2 and nginx3
```bash
kubectl label pod nginx(1..3} env=dev --overwrite
``` 

Add label with key-value 'feature=frontend' and pods named nginx1, nginx2 and nginx3
```bash
kubectl label pod nginx(1..3} feature=frontend 
```

4. Get count of selected

```bash
kubectl get all -l key1=value1 --no-headers | wc -l
```

5. Use multiple selectors
```bash
kubectl get all -l key1=value1,key2=value2
# note: no space between two key value pairs
```

# **Annotations**

We can use Kubernetes annotations to attach arbitrary non-identifying metadata to objects. Clients such as tools and libraries can retrieve this metadata.

## **Attaching metadata to objects**

You can use either labels or annotations to attach metadata to Kubernetes objects. Labels can be used to select objects and to find collections of objects that satisfy certain conditions. In contrast, annotations are not used to identify and select objects. The metadata in an annotation can be small or large, structured or unstructured, and can include characters not permitted by labels.

Annotations, like labels, are key/value maps:

```yaml
"metadata": {
  "annotations": {
    "key1" : "value1",
    "key2" : "value2"
  }
}
```
## **Imperitive Commands**

Add annotations with key-value 'name=webapp' and pods named nginx1, nginx2 and nginx3
```bash
kubectl annotate pod nginx{1..3} name=webapp
```

Verify the pods that have been annotated correctly
```bash
kubectl describe po nginx{1..3} | grep -i annotations
```

Remove the annotations on the pods and verify
```bash
kubectl annotate pod nginx{1..3} name-
kubectl describe po nginx{1..3} | grep -i annotations
```

Remove all the pods that we created so far
```bash
kubectl delete po --all
```

# **Deployments: Rollout & Rollback**

## **Rollout**

Rollout of Deployment allow update to take place.

### **Deployment Strategy**



1. **`RollingUpdate` Strategy (Default)**

A rolling deployment is the default deployment strategy in Kubernetes. It replaces the existing version of pods with a new version, updating pods slowly one by one, without cluster downtime. 

The rolling update uses a readiness probe to check if a new pod is ready, before starting to scale down pods with the old version. If there is a problem, you can stop an update and roll it back, without stopping the entire cluster. 

To perform a rolling update, simply update the image of your pods using kubectl set image. This will automatically trigger a rolling update.

To refine your deployment strategy, change the parameters in the `spec:strategy` section of your manifest file. There are two optional parameters — `maxSurge` and `maxUnavailable`: 

* `MaxSurge`: specifies the maximum number of pods the Deployment is allowed to create at one time. You can specify this as a whole number (e.g. 5), or as a percentage of the total required number of pods (e.g. 10%, always rounded up to the next whole number). If you do not set MaxSurge, the implicit, default value is 25%.

* `MaxUnavailable`: specifies the maximum number of pods that are allowed to be unavailable during the rollout. Like MaxSurge, you can define it as an absolute number or a percentage. 

_At least one of these parameters must be larger than zero_. By changing the values of these parameters, you can define other deployment strategies, as shown below.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
spec:
  minReadySeconds: 20
  progressDeadlineSeconds: 600
  replicas: 4
  selector:
    matchLabels:
      name: webapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: webapp
    spec:
      containers:
      - image: kodekloud/webapp-color:v2
        imagePullPolicy: IfNotPresent
        name: simple-webapp
        ports:
        - containerPort: 8080
          protocol: TCP
```

2. **`Recreate` Strategy**

This is a basic deployment pattern which simply shuts down all the old pods and replaces them with new ones. You define it by setting the `spec:strategy:type` section of your manifest to `Recreate`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
spec:
  minReadySeconds: 20
  progressDeadlineSeconds: 600
  replicas: 4
  selector:
    matchLabels:
      name: webapp
  strategy:
    type: Recreate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: webapp
    spec:
      containers:
      - image: kodekloud/webapp-color:v2
        imagePullPolicy: IfNotPresent
        name: simple-webapp
        ports:
        - containerPort: 8080
          protocol: TCP
```

The Recreate strategy can result in downtime, because old pods are deleted before ensuring that new pods are rolled out with the new version of the application.

--- 
The difference between the recreate and rolling update strategies can also be seen when you view the deployments in detail around the `kubectl describe deployment` command to see the detailed information regarding the deployments.

You will notice when the _*recreate strategy*_ was used, the events indicate that the old `replicaSet` was scaled down to zero and then the new `replicaSet` scaled up to desired number.

However, when the _*rolling update strategy*_ was used, the old replica set was scaled down one at a time, simultaneously scaling up the new replica set, one at a time.

---

### **Creating a deployment**

```bash
kubectl create deployment <deployment-name> --image=<image-name>
----------------------------------------------------------------
deployment.apps/deployment-name created
```
### **Get rollout status**

```bash
kubectl rollout status deployment/<deployment-name>
--------------------------------------------------
Waiting for deployment "<deployment-name>" rollout to finish: 0 of 1 updated replicas are available...
deployment "<deployment-name>" successfully rolled out
```

### **Get rollout history**

```bash
kubectl rollout history deployment/<deployment-name>
---------------------------------------------------
deployment.extensions/<deployment-name>
REVISION CHANGE-CAUSE
1     <none>
```

- Using the `--revision` flag:

Here the revision 1 is the first version where the deployment was created.

You can check the status of each revision individually by using the `--revision` flag:

```bash
kubectl rollout history deployment <deployment-name> --revision=1
-----------------------------------------------------------------
deployment.extensions/<deployment-name> with revision #1
 
Pod Template:
 Labels:    app=<app-name>    pod-template-hash=6454457cdb
 Containers:  <container-name>:  Image:   <image-name>:1.16
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>
```

- Using the `--record` flag:

You would have noticed that the **change-cause** field is empty in the rollout history output. We can use the `--record` flag to save the command used to create/update a deployment against the revision number.

Once we make the necessary changes, we run the `kubectl apply` command to apply the changes.

A new rollout is triggered and a new revision of the deployment is created, but some changes can also be done by some cli commands like changing the image of your application:

But remember, doing it this way will result in the deployment definition file having a different configuration.

So you must be careful when using the same definition file to make changes in the future.

```bash
kubectl set image deployment <deployment-name> <container-name>=<image-name>:1.17 --record
------------------------------------------------------------------------------------------
deployment.extensions/<deployment-name> image updated
```

 
```bash
kubectl rollout history deployment <deployment-name>
----------------------------------------------------
deployment.extensions/<deployment-name>
 
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment <deployment-name> <container-name>=<image-name>:1.17 --record

```
You can now see that the **change-cause** is recorded for the revision 2 of this deployment.

Let's make some more changes. In the example below, we are editing the deployment and changing the image from <image-name>:1.17 to <image-name>:latest while making use of the `--record` flag.

```bash
kubectl edit deployment <deployment-name> --record
----------------------------------------------------
deployment.extensions/<deployment-name> edited
```
 
```bash
kubectl rollout history deployment <deployment-name>
----------------------------------------------------
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment <deployment-name>  <container-name>=<image-name>:1.17 --record
3     kubectl edit deployments. <deployment-name> --record=true
```
 
```bash
kubectl rollout history deployment <deployment-name> --revision=3
-----------------------------------------------------------------
deployment.extensions/<deployment-name> with revision #3
Pod Template: Labels:    app=<app-name>
    pod-template-hash=df6487dc Annotations: kubernetes.io/change-cause: kubectl edit deployments. <deployment-name> --record=true
 
 Containers:
  <container-name>:
  Image:   <image-name>:latest
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>
```


## **Rollback**

```bash
kubectl rollout undo deployment/<deployment-name>
```

# **Job**

A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. 
- As pods successfully complete, the Job tracks the successful completions. 
- When a specified number of successful completions is reached, the task (ie, Job) is complete. 
- Deleting a Job will clean up the Pods it created. 
- Suspending a Job will delete its active Pods until the Job is resumed again.

A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is deleted (for example due to a node hardware failure or a node reboot).

We can also use a Job to run multiple Pods in parallel.

If you want to run a Job (either a single task, or several in parallel) on a schedule, see [CronJob](#cronjob).

## **Running a Job**

Here is an example Job config. It adds two numbers.

```yaml
apiVersion: batch/v1 
kind: Job
metadata:
  name: math-add-job
spec:
  # Specifies how many pods to create for this job
  completions: 3
  # Specifies how many pods should run in parallel
  parallelism: 3
  template:
    # Copy from POD .spec
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ["expr", "3", "+", "2"]
      restartPolicy: OnFailure
```

We can get basic job definition file with cli using:
```bash
kubectl create job <job-name> --image <image-name> --dry-run=client -o yaml > job.yaml
```


To list all the Pods that belong to a Job in a machine readable form, you can use a command like this:

```bash
pods=$(kubectl get pods --selector=job-name=pi --output=jsonpath='{.items[*].metadata.name}')

echo $pods
------------------------------------------------
pi-5rwd7
```
The output is similar to this:

View the standard output of one of the pods:

```bash
kubectl logs $pods/<pod-name>

```

Deleting a job terminates all the pods under it. To delete a job without deleting pods use:
```bash
kubectl delete jobs/old --cascade=orphan
```
## **Writing a Job spec**

As with all other Kubernetes config, a Job needs `apiVersion`, `kind`, and `metadata` fields.
A Job also needs a `.spec`

### **Pod Template**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: math-add-pod
spec:
  containers:
    - name: math-add
      image: ubuntu
      command: ["expr", "3", "+", "2"]
  restartPolicy: OnFailure
```


The `.spec.template` is the only required field of the `.spec`.

The `.spec.template` of Job definition is a [pod](#pod-template) `.spec` section. It has exactly the same schema as a Pod `.spec` section.

In addition to required fields for a Pod, a pod template in a Job must specify appropriate labels (see [pod selector](#pod-selector)) and an appropriate restart policy.

>Only a [RestartPolicy](../Observability/observability.md#container-restart-policy) equal to `Never` or `OnFailure` is allowed.

### **Pod Selector**

The `.spec.selector` field is optional. In almost all cases you should not specify it. See section [specifying your own pod selector](https://kubernetes.io/docs/concepts/workloads/controllers/job/#specifying-your-own-pod-selector).

### **Parallel execution for Jobs**

There are three main types of task suitable to run as a Job:

1. Non-parallel Jobs
    - normally, only one Pod is started, unless the Pod fails.
    - the Job is complete as soon as its Pod terminates successfully.
2. Parallel Jobs with a *fixed completion count*:
    - specify a non-zero positive value for `.spec.completions`.
    - the Job represents the overall task, and is complete when there are `.spec.completions` successful Pods.
    - when using `.spec.completionMode="Indexed"`, each Pod gets a different index in the range 0 to `.spec.completions-1`.
3. Parallel Jobs with a *work queue*:
    - do not specify `.spec.completions`, default to `.spec.parallelism`.
    - the Pods must coordinate amongst themselves or an external service to determine what each should work on. For example, a Pod might fetch a batch of up to N items from the work queue.
    - each Pod is independently capable of determining whether or not all its peers are done, and thus that the entire Job is done.
    - when *any* Pod from the Job terminates with success, no new Pods are created.
    - once at least one Pod has terminated with success and all Pods are terminated, then the Job is completed with success.
      - once any Pod has exited with success, no other Pod should still be doing any work for this task or writing any output. They should all be in the process of exiting.

For a *non-parallel* Job, you can leave both `.spec.completions` and `.spec.parallelism` unset. When both are unset, both are defaulted to 1.

For a *fixed completion count* Job, you should set `.spec.completions` to the number of completions needed. You can set `.spec.parallelism`, or leave it unset and it will default to 1.

For a *work queue* Job, you must leave `.spec.completions` unset, and set `.spec.parallelism` to a non-negative integer.

For more information about how to make use of the different types of job, see the [job patterns](#job-patterns) section.

### **Controlling parallelism**

The requested parallelism (`.spec.parallelism`) can be set to any non-negative value. If it is unspecified, it defaults to 1. If it is specified as 0, then the Job is effectively paused until it is increased.

Actual parallelism (number of pods running at any instant) may be more or less than requested parallelism, for a variety of reasons:

- For *fixed completion count* Jobs, the actual number of pods running in parallel will not exceed the number of remaining completions. Higher values of `.spec.parallelism` are effectively ignored.
- For *work queue* Jobs, no new Pods are started after any Pod has succeeded -- remaining Pods are allowed to complete, however.
- If the Job [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) has not had time to react.
- If the Job controller failed to create Pods for any reason (lack of `ResourceQuota`, lack of permission, etc.), then there may be fewer pods than requested.
- The Job controller may throttle new Pod creation due to excessive previous pod failures in the same Job.
- When a Pod is gracefully shut down, it takes time to stop.

### **Completion mode**

Jobs with *fixed completion count* - that is, jobs that have non null `.spec.completions` - can have a completion mode that is specified in `.spec.completionMode`:

- `NonIndexed` (default): the Job is considered complete when there have been `.spec.completions` successfully completed Pods. In other words, each Pod completion is homologous to each other. Note that Jobs that have null `.spec.completions` are implicitly `NonIndexed`.
- `Indexed`: the Pods of a Job get an associated completion index from 0 to `.spec.completions-1`. The index is available through three mechanisms:
    - The Pod annotation `batch.kubernetes.io/job-completion-index`.
    - As part of the Pod hostname, following the pattern `$(job-name)-$(index)`. When you use an Indexed Job in combination with a [Service](https://kubernetes.io/docs/concepts/services-networking/service/), Pods within the Job can use the deterministic hostnames to address each other via DNS.
    - From the containarized task, in the environment variable `JOB_COMPLETION_INDEX`.
    
    The Job is considered complete when there is one successfully completed Pod for each index. For more information about how to use this mode, see [Indexed Job for Parallel Processing with Static Work Assignment](https://kubernetes.io/docs/tasks/job/indexed-parallel-processing-static/). Note that, although rare, more than one Pod could be started for the same index, but only one of them will count towards the completion count.

## **Handling Pod and container failures**

A container in a Pod may fail for a number of reasons, such as because the process in it exited with a non-zero exit code, or the container was killed for exceeding a memory limit, etc. If this happens, and the `.spec.template.spec.restartPolicy = "OnFailure"`, _then the Pod stays on the node, but the container is re-run_. Therefore, your program needs to handle the case when it is restarted locally, or else specify `.spec.template.spec.restartPolicy = "Never"`. See [pod lifecycle](../Observability/observability.md#container-restart-policy) for more information on `restartPolicy`.

An entire Pod can also fail, for a number of reasons, such as when the pod is kicked off by the node (node is upgraded, rebooted, deleted, etc.), or if a container of the Pod fails and the `.spec.template.spec.restartPolicy = "Never"`. When a Pod fails, then the Job controller starts a new Pod. This means that your application needs to handle the case when it is restarted in a new pod. In particular, it needs to handle temporary files, locks, incomplete output and the like caused by previous runs.

Note that even if you specify `.spec.parallelism = 1` and `.spec.completions = 1` and `.spec.template.spec.restartPolicy = "Never"`, the same program may sometimes be started twice.

If you do specify `.spec.parallelism` and `.spec.completions` both greater than 1, then there may be multiple pods running at once. Therefore, your pods must also be tolerant of concurrency.


### **Pod backoff failure policy**

There are situations where you want to fail a Job after some amount of retries due to a logical error in configuration etc. To do so, set `.spec.backoffLimit` to specify the number of retries before considering a Job as failed. **_The back-off limit is set by default to 6_**. Failed Pods associated with the Job are recreated by the Job controller with an exponential back-off delay (10s, 20s, 40s ...) capped at six minutes. The back-off count is reset when a Job's Pod is deleted or successful without any other Pods for the Job failing around that time.

> **Note:** 
> 
> If your job has `restartPolicy = "OnFailure"`, keep in mind that your Pod running the Job will be terminated once the job backoff limit has been reached. This can make debugging the Job's executable more difficult. We suggest setting `restartPolicy = "Never"` when debugging the Job or using a logging system to ensure output from failed Jobs is not lost inadvertently.

## **Job termination and cleanup**

When a Job completes, no more Pods are created, but the Pods are [usually](#pod-backoff-failure-policy) not deleted either. Keeping them around allows you to still view the logs of completed pods to check for errors, warnings, or other diagnostic output. The job object also remains after it is completed so that you can view its status. _It is up to the user to delete old jobs after noting their status_. Delete the job with `kubectl` (e.g. `kubectl delete jobs/<job-name>` or `kubectl delete -f ./job.yaml`). When you delete the job using `kubectl`, all the pods it created are deleted too.

By default, a Job will run uninterrupted unless a Pod fails (`restartPolicy=Never`) or a Container exits in error (`restartPolicy=OnFailure`), at which point the Job defers to the `.spec.backoffLimit` described above. Once `.spec.backoffLimit` has been reached the Job will be marked as failed and any running Pods will be terminated.

Another way to terminate a Job is by setting an active deadline. Do this by setting the `.spec.activeDeadlineSeconds` field of the Job to a number of seconds. The `activeDeadlineSeconds` applies to the duration of the job, no matter how many Pods are created. Once a Job reaches `activeDeadlineSeconds`, all of its running Pods are terminated and the Job status will become `type: Failed` with `reason: DeadlineExceeded`.

Note that a Job's `.spec.activeDeadlineSeconds` takes precedence over its `.spec.backoffLimit`. Therefore, a Job that is retrying one or more failed Pods will not deploy additional Pods once it reaches the time limit specified by `activeDeadlineSeconds`, even if the `backoffLimit` is not yet reached.

Example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-timeout
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

Note that both the Job spec and the [Pod template spec](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#detailed-behavior) within the Job have an `activeDeadlineSeconds` field. Ensure that you set this field at the proper level.

Keep in mind that the `restartPolicy` applies to the Pod, and not to the Job itself: there is no automatic Job restart once the Job status is `type: Failed`. That is, the Job termination mechanisms activated with `.spec.activeDeadlineSeconds` and `.spec.backoffLimit` result in a permanent Job failure that requires manual intervention to resolve.

## **Clean up finished jobs automatically**

Finished Jobs are usually no longer needed in the system. Keeping them around in the system will put pressure on the API server. If the Jobs are managed directly by a higher level controller, such as [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/), the Jobs can be cleaned up by CronJobs based on the specified capacity-based cleanup policy.

### **TTL mechanism for finished Jobs**

Another way to clean up finished Jobs (either `Complete` or `Failed`) automatically is to use a TTL mechanism provided by a [TTL controller](https://kubernetes.io/docs/concepts/workloads/controllers/ttlafterfinished/) for finished resources, by specifying the `.spec.ttlSecondsAfterFinished` field of the Job.

When the TTL controller cleans up the Job, it will delete the Job cascadingly, i.e. delete its dependent objects, such as Pods, together with the Job. Note that when the Job is deleted, its lifecycle guarantees, such as finalizers, will be honored.

For example:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: 100
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

The Job `pi-with-ttl` will be eligible to be automatically deleted, `100` seconds after it finishes.

If the field is set to `0`, the Job will be eligible to be automatically deleted immediately after it finishes. If the field is unset, this Job won't be cleaned up by the TTL controller after it finishes.

> **Note:**
> It is recommended to set `ttlSecondsAfterFinished` field because unmanaged jobs (Jobs that you created directly, and not indirectly through other workload APIs such as CronJob) have a default deletion policy of `orphanDependents` causing Pods created by an unmanaged Job to be left around after that Job is fully deleted. Even though the [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) eventually [garbage collects](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-garbage-collection) the Pods from a deleted Job after they either fail or complete, sometimes those lingering pods may cause cluster performance degradation or in worst case cause the cluster to go offline due to this degradation.
>
> You can use [LimitRanges](https://kubernetes.io/docs/concepts/policy/limit-range/) and [ResourceQuotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/) to place a cap on the amount of resources that a particular namespace can consume.

## **Job patterns**

The Job object can be used to support reliable parallel execution of Pods. The Job object is not designed to support closely-communicating parallel processes, as commonly found in scientific computing. It does support parallel processing of a set of independent but related *work items*. These might be emails to be sent, frames to be rendered, files to be transcoded, ranges of keys in a NoSQL database to scan, and so on.

In a complex system, there may be multiple different sets of work items. Here we are just considering one set of work items that the user wants to manage together — a *batch job*.

There are several different patterns for parallel computation, each with strengths and weaknesses. The tradeoffs are:

- One Job object for each work item, vs. a single Job object for all work items. The latter is better for large numbers of work items. The former creates some overhead for the user and for the system to manage large numbers of Job objects.
- Number of pods created equals number of work items, vs. each Pod can process multiple work items. The former typically requires less modification to existing code and containers. The latter is better for large numbers of work items, for similar reasons to the previous bullet.
- Several approaches use a work queue. This requires running a queue service, and modifications to the existing program or container to make it use the work queue. Other approaches are easier to adapt to an existing containerised application.

The tradeoffs are summarized here, with columns 2 to 4 corresponding to the above tradeoffs. The pattern names are also links to examples and more detailed description.

| Pattern                                 | Single Job object | Fewer pods than work items? | Use app unmodified? |
| --------------------------------------- | ----------------- | --------------------------- | ------------------- |
| [Queue with Pod Per Work Item](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/)            | ✓                 |                             | sometimes           |
| [Queue with Variable Pod Count](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)           | ✓                 | ✓                           |                     |
| [Indexed Job with Static Work Assignment](https://kubernetes.io/docs/tasks/job/indexed-parallel-processing-static/) | ✓                 |                             | ✓                   |
| [Job Template Expansion](https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/)                  |                   |                             | ✓                   |

When you specify completions with `.spec.completions`, each Pod created by the Job controller has an identical `.spec`. This means that all pods for a task will have the same command line and the same image, the same volumes, and (almost) the same environment variables. These patterns are different ways to arrange for pods to work on different things.

This table shows the required settings for `.spec.parallelism` and `.spec.completions` for each of the patterns. Here, `W` is the number of work items.

| Pattern                                 |` .spec.completions` | `.spec.parallelism` |
| --------------------------------------- | ----------------- | ----------------- |
| [Queue with Pod Per Work Item](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/)            | W                 | any               |
| [Queue with Variable Pod Count](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)           | null              | any               |
| [Indexed Job with Static Work Assignment](https://kubernetes.io/docs/tasks/job/indexed-parallel-processing-static/) | W                 | any               |
| [Job Template Expansion](https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/)                  | 1                 | should be 1       |


# **CronJob**

A CronJob creates Jobs on a repeating schedule.

One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.

> **Caution:**
> 
> All CronJob schedule: times are based on the timezone of the kube-controller-manager.
> 
> If your control plane runs the kube-controller-manager in Pods or bare containers, the timezone set for the kube-controller-manager container determines the timezone that the cron job controller uses.

CronJobs are meant for performing regular scheduled actions such as backups, report generation, and so on. Each of those tasks should be configured to recur indefinitely (for example: once a day / week / month); you can define the point in time within that interval when the job should start.

Example
This example CronJob manifest prints the current time and a hello message every minute:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
([Running Automated Tasks with a CronJob](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/) takes you through this example in more detail).

## **Cron schedule syntax**

    # ┌───────────── minute (0 - 59)
    # │ ┌───────────── hour (0 - 23)
    # │ │ ┌───────────── day of the month (1 - 31)
    # │ │ │ ┌───────────── month (1 - 12)
    # │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
    # │ │ │ │ │                                   7 is also Sunday on some systems)
    # │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
    # │ │ │ │ │
    # * * * * *

| Entry                  | Description                                                | Equivalent to |
| ---------------------- | ---------------------------------------------------------- | ------------- |
| @yearly (or @annually) | Run once a year at midnight of 1 January                   | 0 0 1 1 *     |
| @monthly               | Run once a month at midnight of the first day of the month | 0 0 1 * *     |
| @weekly                | Run once a week at midnight on Sunday morning              | 0 0 * * 0     |
| @daily (or @midnight)  | Run once a day at midnight                                 | 0 0 * * *     |
| @hourly                | Run once an hour at the beginning of the hour              | 0 * * * *     |


For example, the line below states that the task must be started every Friday at midnight, as well as on the 13th of each month at midnight:

`0 0 13 * 5`


We can get basic cronJob definition file with cli using:
```bash
kubectl create cronjob <cron=job-name> --image <image-name> --schedule "0 0 13 * 5" --dry-run=client -o yaml > cron-job.yaml
```


# **Refernces**

1. [5 Kubernetes Deployment Strategies: Roll Out Like the Pros](https://spot.io/resources/kubernetes-autoscaling/5-kubernetes-deployment-strategies-roll-out-like-the-pros)
2. [k8 Jobs Docs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
3. [K8 CronJob Docs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)