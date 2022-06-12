# **Admission Controllers**
An admission controller is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but after the request is authenticated and authorized. 

Admission controllers limit requests to create, delete, modify objects or connect to proxy. They do not limit requests to read objects.

There are two types of admission controllers:
- Mutating admission controllers are those that can change the request.
- Validating admission controllers are those that can validate the request and allow or denied.

Generally mutating admission controllers are invoked first, followed by validating that admission controllers. This is so that any change made by the mutating admission controller can be considered during the validation process.


### Which plugins are enabled by default?
To see which admission plugins are enabled:
```
kube-apiserver -h | grep enable-admission-plugins
```

In the current version, the default ones are:
```
CertificateApproval, CertificateSigning, CertificateSubjectRestriction, DefaultIngressClass, DefaultStorageClass, DefaultTolerationSeconds, LimitRanger, MutatingAdmissionWebhook, NamespaceLifecycle, PersistentVolumeClaimResize, Priority, ResourceQuota, RuntimeClass, ServiceAccount, StorageObjectInUseProtection, TaintNodesByCondition, ValidatingAdmissionWebhook
```

> Note that the `NamespaceExists` and `NamespaceAutoProvision` admission controllers are *deprecated* and now *replaced by* `NamespaceLifecycle` admission controller.
>
>The `NamespaceLifecycle` admission controller will make sure that requests to a non-existent namespace is rejected and that the default namespaces such as `default`, `kube-system` and `kube-public` cannot be deleted.

Since the kube-apiserver is running as pod you can check the process to see enabled and disabled plugins.

```bash
ps -ef | grep kube-apiserver | grep admission-plugins
```

**Loaction of `kube-apiserver.yaml`** : `/etc/kubernetes/manifests/kube-apiserver.yaml`

## **Custom Admission Controller**

There are two special admission controllers available: 
- mutating admission webhook 
- validating admission webhook.

We can configure these Webhooks to point to a `server` that's hosted either within the `Kubernetes` cluster or outside it, and our server will have our own admission webhook service running with our own code and logic. 

After a request goes through all the built in admission controllers, it hits the the webhook and it makes a call to the admission web server by passing an `admission review object` in a `JSON` format. This object has all the details about the request, such as the user that made the request and the type of operation the user is trying to perform and on what object and details about the object itself on receiving the request.

The admission webhook server responds with an admission review object with a result of whether the request is allowed or not.

If the `allowed field` in the response is set to `True`, then the request is `allowed` and if it's set to `False`, it is `rejected`.

```yaml
apiVersion: admissionregistration.k8s.io/v1beta1
kind: MutatingWebhookConfiguration
metadata:
  name: demo-webhook
webhooks:
  - name: webhook-server.webhook-demo.svc
    clientConfig:
      service:
        name: webhook-server
        namespace: webhook-demo
        path: "/mutate"
      caBundle: data
    rules:
      - operations: [ "CREATE" ]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
  - name: webhook-client.webhook-demo.svc
    clientConfig:
      service:
        url: url
      caBundle: data
    rules:
      - operations: [ "CREATE" ]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
```