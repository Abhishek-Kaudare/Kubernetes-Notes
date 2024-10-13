| Resource                   | Shortform |
|:--------------------------:|:---------:|
| pod                        | po        |
| replicaset                 | rs        |
| replicationcontrollers     | rc        |
| namespace                  | ns        |
| configmap                  | cm        |
| node                       | no        |
| resourcequotas             | quota     |
| serviceAccount             | sa        |
| deployments                | deploy    |
| service                    | svc       |
| ingresses                  | ing       |
| endpoints                  | ep        |
| limitranges                | limits    |
| networkPolicies            | netpol    |
| persistentVolumes          | pv        |
| persistentVolumeClaims     | pvc       |
| certificatesigningrequests | csr       |
| componentstatuses          | cs        |
| daemonsets                 | ds        |
| events                     | ev        |
| horizontalpodautoscalers   | hpa       |
| poddisruptionbudgets       | pdb       |
| podsecuritypolicies        | psp       |

# Important Api Resources

| NAME |                              SHORTNAMES   | APIVERSION                             | NAMESPACED | KIND |
| :---: | :---: | :---: | :---: |:---------:|
| componentstatuses |                 cs           | v1                                     | false | ComponentStatus |
| configmaps |                        cm           | v1                                     | true | ConfigMap |
| endpoints |                         ep           | v1                                     | true | Endpoints |
| events |                            ev           | v1                                     | true | Event |
| limitranges |                       limits       | v1                                     | true | LimitRange |
| namespaces |                        ns           | v1                                     | false | Namespace |
| nodes |                             no           | v1                                     | false | Node |
| persistentvolumeclaims |            pvc          | v1                                     | true | PersistentVolumeClaim |
| persistentvolumes |                 pv           | v1                                     | false | PersistentVolume |
| pods |                              po           | v1                                     | true | Pod |
| replicationcontrollers |            rc           | v1                                     | true | ReplicationController |
| resourcequotas |                    quota        | v1                                     | true | ResourceQuota |
| secrets |                                        | v1                                     | true | Secret |
| serviceaccounts |                   sa           | v1                                     | true | ServiceAccount |
| services |                          svc          | v1                                     | true | Service |
| customresourcedefinitions |         crd,crds     | apiextensions.k8s.io/v1                | false | CustomResourceDefinition |
| daemonsets |                        ds           | apps/v1                                | true | DaemonSet |
| deployments |                       deploy       | apps/v1                                | true | Deployment |
| replicasets |                       rs           | apps/v1                                | true | ReplicaSet |
| cronjobs |                          cj           | batch/v1                               | true | CronJob |
| jobs |                                           | batch/v1                               | true | Job |
| certificatesigningrequests |        csr          | certificates.k8s.io/v1                 | false | CertificateSigningRequest |
| leases |                                         | coordination.k8s.io/v1                 | true | Lease |
| endpointslices |                                 | discovery.k8s.io/v1                    | true | EndpointSlice |
| events |                            ev           | events.k8s.io/v1                       | true | Event |
| ingresses |                         ing          | networking.k8s.io/v1                   | true | Ingress |
| networkpolicies |                   netpol       | networking.k8s.io/v1                   | true | NetworkPolicy |
| podsecuritypolicies |               psp          | policy/v1beta1                         | false | PodSecurityPolicy |
| clusterrolebindings |                            | rbac.authorization.k8s.io/v1           | false | ClusterRoleBinding |
| clusterroles |                                   | rbac.authorization.k8s.io/v1           | false | ClusterRole |
| rolebindings |                                   | rbac.authorization.k8s.io/v1           | true | RoleBinding |
| roles |                                          | rbac.authorization.k8s.io/v1           | true | Role |

# All Api Resources

| NAME |                              SHORTNAMES   | APIVERSION                             | NAMESPACED | KIND |
| :---: | :---: | :---: | :---: |
| componentstatuses |                 cs           | v1                                     | false | ComponentStatus |
| configmaps |                        cm           | v1                                     | true | ConfigMap |
| endpoints |                         ep           | v1                                     | true | Endpoints |
| events |                            ev           | v1                                     | true | Event |
| limitranges |                       limits       | v1                                     | true | LimitRange |
| namespaces |                        ns           | v1                                     | false | Namespace |
| nodes |                             no           | v1                                     | false | Node |
| persistentvolumeclaims |            pvc          | v1                                     | true | PersistentVolumeClaim |
| persistentvolumes |                 pv           | v1                                     | false | PersistentVolume |
| pods |                              po           | v1                                     | true | Pod |
| podtemplates |                                   | v1                                     | true | PodTemplate |
| replicationcontrollers |            rc           | v1                                     | true | ReplicationController |
| resourcequotas |                    quota        | v1                                     | true | ResourceQuota |
| secrets |                                        | v1                                     | true | Secret |
| serviceaccounts |                   sa           | v1                                     | true | ServiceAccount |
| services |                          svc          | v1                                     | true | Service |
| mutatingwebhookconfigurations |                  | admissionregistration.k8s.io/v1        | false | MutatingWebhookConfiguration |
| validatingwebhookconfigurations |                | admissionregistration.k8s.io/v1        | false | ValidatingWebhookConfiguration |
| customresourcedefinitions |         crd,crds     | apiextensions.k8s.io/v1                | false | CustomResourceDefinition |
| apiservices |                                    | apiregistration.k8s.io/v1              | false | APIService |
| controllerrevisions |                            | apps/v1                                | true | ControllerRevision |
| daemonsets |                        ds           | apps/v1                                | true | DaemonSet |
| deployments |                       deploy       | apps/v1                                | true | Deployment |
| replicasets |                       rs           | apps/v1                                | true | ReplicaSet |
| statefulsets |                      sts          | apps/v1                                | true | StatefulSet |
| tokenreviews |                                   | authentication.k8s.io/v1               | false | TokenReview |
| localsubjectaccessreviews |                      | authorization.k8s.io/v1                | true | LocalSubjectAccessReview |
| selfsubjectaccessreviews |                       | authorization.k8s.io/v1                | false | SelfSubjectAccessReview |
| selfsubjectrulesreviews |                        | authorization.k8s.io/v1                | false | SelfSubjectRulesReview |
| subjectaccessreviews |                           | authorization.k8s.io/v1                | false | SubjectAccessReview |
| horizontalpodautoscalers |          hpa          | autoscaling/v2                         | true | HorizontalPodAutoscaler |
| cronjobs |                          cj           | batch/v1                               | true | CronJob |
| jobs |                                           | batch/v1                               | true | Job |
| certificatesigningrequests |        csr          | certificates.k8s.io/v1                 | false | CertificateSigningRequest |
| leases |                                         | coordination.k8s.io/v1                 | true | Lease |
| endpointslices |                                 | discovery.k8s.io/v1                    | true | EndpointSlice |
| events |                            ev           | events.k8s.io/v1                       | true | Event |
| flowschemas |                                    | flowcontrol.apiserver.k8s.io/v1beta2   | false | FlowSchema |
| prioritylevelconfigurations |                    | flowcontrol.apiserver.k8s.io/v1beta2   | false | PriorityLevelConfiguration |
| helmchartconfigs |                               | helm.cattle.io/v1                      | true | HelmChartConfig |
| helmcharts |                                     | helm.cattle.io/v1                      | true | HelmChart |
| addons |                                         | k3s.cattle.io/v1                       | true | Addon |
| nodes |                                          | metrics.k8s.io/v1beta1                 | false | NodeMetrics |
| pods |                                           | metrics.k8s.io/v1beta1                 | true | PodMetrics |
| ingressclasses |                                 | networking.k8s.io/v1                   | false | IngressClass |
| ingresses |                         ing          | networking.k8s.io/v1                   | true | Ingress |
| networkpolicies |                   netpol       | networking.k8s.io/v1                   | true | NetworkPolicy |
| runtimeclasses |                                 | node.k8s.io/v1                         | false | RuntimeClass |
| poddisruptionbudgets |              pdb          | policy/v1                              | true | PodDisruptionBudget |
| podsecuritypolicies |               psp          | policy/v1beta1                         | false | PodSecurityPolicy |
| clusterrolebindings |                            | rbac.authorization.k8s.io/v1           | false | ClusterRoleBinding |
| clusterroles |                                   | rbac.authorization.k8s.io/v1           | false | ClusterRole |
| rolebindings |                                   | rbac.authorization.k8s.io/v1           | true | RoleBinding |
| roles |                                          | rbac.authorization.k8s.io/v1           | true | Role |
| priorityclasses |                   pc           | scheduling.k8s.io/v1                   | false | PriorityClass |
| csidrivers |                                     | storage.k8s.io/v1                      | false | CSIDriver |
| csinodes |                                       | storage.k8s.io/v1                      | false | CSINode |
| csistoragecapacities |                           | storage.k8s.io/v1beta1                 | true | CSIStorageCapacity |
| storageclasses |                    sc           | storage.k8s.io/v1                      | false | StorageClass |
| volumeattachments |                              | storage.k8s.io/v1                      | false | VolumeAttachment |
| ingressroutes |                                  | traefik.containo.us/v1alpha1           | true | IngressRoute |
| ingressroutetcps |                               | traefik.containo.us/v1alpha1           | true | IngressRouteTCP |
| ingressrouteudps |                               | traefik.containo.us/v1alpha1           | true | IngressRouteUDP |
| middlewares |                                    | traefik.containo.us/v1alpha1           | true | Middleware |
| middlewaretcps |                                 | traefik.containo.us/v1alpha1           | true | MiddlewareTCP |
| serverstransports |                              | traefik.containo.us/v1alpha1           | true | ServersTransport |
| tlsoptions |                                     | traefik.containo.us/v1alpha1           | true | TLSOption |
| tlsstores |                                      | traefik.containo.us/v1alpha1           | true | TLSStore |
| traefikservices |                                | traefik.containo.us/v1alpha1           | true | TraefikService |
