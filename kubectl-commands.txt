--- Pods ----------------------------------------------------
# create a pod/replcationcontroller/replicaset
kubectl create -f pod-definition.yml

# get running node/pod/replcationcontroller/replicaset
kubectl get pods

# delete a running pod/replcationcontroller/replicaset with name
kubectl delete pod <<pod_name>>

# get info about a new pod
kubectl describe pod <<pod_name>>

# edit pod config
kubectl edit pod <<pod_name>>

# check pods in detail
kubectl get pods --all-namespaces -o wide

# check logs of a container in an app/pod
kubectl logs webapp-2 -c simple-webapp

# streamline logs
kubectl logs webapp-2 -f

# check logs of a pod at a particular namespace
kubectl -n elastic-stack exec -it app cat /log/app.log

# get inside an app 
kubectl -n elastic-stack exec -it app cat /log/app.log

# get existing pod config as yaml/file
kubectl get pod app -n elastic-stack -o yaml > app.yaml

# who is running the pod
kubectl exec ubuntu-sleeper -- whoami

# to convert a secret to base64
echo -n 'string' | base64

# check secrets inside the container
ls /opt/app-secret-volumes

# or
kubectl exec -ti webapp-pod -- cat /opt/webapp-mysql/kube-files/secret-data.yaml

# extract secret
cat /opt/app-secret-volumes/{{secret_name}}

# find pods by label
kubectl get pods -l name=payroll

# show labels
kubectl get pods --show-labels

# to get the image used by a scheduler pod
kubectl -n kube-system describe pod kube-sceduler-master | grep -i image

# delete a pod once created (and exited immidiately)
kubectl run --rm -it --image=<image> <podname> -- sh

# check env vars in a pod
kubectl exec <pod> -- env

# pod consuming max resouces
kubectl top pod --sort-by='cpu'

# explain pods
kubectl explain pods --recursive | less

# create yaml easily
kubectl run pod my-busybox --image=busybox --namespace=dev2406 --command sleep 3600 --dry-run=client -o yaml > busybox.yaml

# get pod's security context
kubectl get pod pod-with-defaults -o yaml | grep -A2 securityContext

# forcefully recreate pods
kubectl apply -f new-defn.yaml --force

--- Replicaset ----------------------------------------------------
# scale the replicaset with more replica
kubectl replace -f replicaset-definition.yml

# another way
kubectl scale --replicas=6 -f replicaset-definition.yml

# get all replicasets
kubectl get replicasets.apps

--- Deployments ----------------------------------------------------
# get all deployments
kubectl get deployments.apps

# get image name
kubectl describe deployments.apps frontend-deployment | grep -i image

# change deployment file and apply
kubectl apply -f deployment-definition.yml

# one line deployment create/edit changes
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine
kubectl scale deployment httpd-frontend --replicas=3 
kubectl expose deployment nginx --port 80

# change image for a deployment
kubectl set image deployment nginx nginx=nginx:1.18

# get status
kubectl get deployments.apps httpd-frontend

# dry run deployment to create a file
kubectl create deployment <<deployment_name>> --image=nginx --dry-run -o yaml > red.yaml

# rollout - status/history
kubectl rollout status deployment nginx
kubectl rollout history deployment nginx
kubectl rollout history deployment nginx --revision=1
kubectl set image deployment nginx nginx=nginx:1.17 --record

--- Contexts ------------------------------------------------------
kubectl config get-contexts                          # display list of contexts 
kubectl config current-context                       # display the current-context
kubectl config use-context my-cluster-name           # set the default context to my-cluster-name

# add a new user to your kubeconf that supports basic auth
kubectl config set-credentials kubeuser/foo.kubernetes.com --username=kubeuser --password=kubepassword

# permanently save the namespace for all subsequent kubectl commands in that context.
kubectl config set-context --current --namespace=ggckad-s2

# set a context utilizing a specific username and namespace.
kubectl config set-context gce --user=cluster-admin --namespace=foo && kubectl config use-context gce

--- Jobs ----------------------------------------------------------
# get job logs
kubectl describe job.batch/throw-dice-job

# to increase chances of completions increase the backoffLimit

--- Namespaces ----------------------------------------------------
# create namespace
kubectl create namespace <<namespace_name>>

# get pods for a specific namespace
kubectl get pods --namespace=prod

# get pods on a specific node 
kubectl get pods --all-namespaces -o wide --field-selector spec.nodeName=node01

# set context for a namespace
kubectl config set-context $(kubectl config current-context) --namespace=dev

# create pod in namespace
kubectl run redis --image=redis --namespace=finance

# find a pod
kubectl get pods --all-namespaces | grep blue

# expose a running pod
kubectl expose pod nginx --port 80 --name=nginx-service

# get namespaces (wc -l count number of lines and --no-headers avoid header)
kubectl get namespace --no-headers | wc -l

# get pods in a namespace
kubectl get pods --namespace=<<namespace_name>>

--- Configs -----------------------------------------------------
# default
kubectl config view

# use another config
kubectl config view --kubeconfig=my-custom-config

# add new context
kubectl config use-context prod-user@production

# check where is config
echo $HOME
ls -a /root
 ....

# check current context
kubectl config current-context

# switch context from a manual new config
kubectl config --kubeconfig=/root/my-kube-config use-context research

--- Authorizations ----------------------------------------------
# get roles
kubectl get roles

# describe role
kubectl describe role <<role_name>>

# describe binding
kubectl describe rolebinding <<binding_name>>

# check permissions
kubectl auth can-i <<create/delete>> <<pods/deployments>> --as dev-user --namespace <<namespace_name>>

# check permission simple
kubectl get pod --as dev-user

# create role
kubectl create role developer --verb=create,delete,list --resource=pods --namespace=default

# create role binding
kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user

# edit role
kubectl edit role developer -n blue

# to what users/groups is a cluster role cluster-admin bound to
kubectl describe clusterrolebinding cluster-admin

# create cluster role
kubectl create clusterrole node-role --resource=nodes --verb=get,watch,list,create,delete

# create cluster-role-binding
kubectl create clusterrolebinding node-role-binding --clusterrole=node-role --user=michelle

# check if it worked
kubectl auth can-i list nodes --as michelle

# get api-resources
kubectl api-resources | sort -u -k1

--- Services ----------------------------------------------------
# create a service
kubectl create -f service-definition.yml

# get services
kubectl get services/svc

# describe services (describe <<type>> <<name>>)
kubectl describe svc kubernetes

# expose a deployment to a file
kubectl expose deployment <<deployment_name>> --name=<<deployment_short_name>> --target-port=<<port_number>> --type=NodePort --port=<<port_number>> --dry-run=client -o yaml > <<deployment_file>>

# change and apply a service <-- declarative
kubectl apply -f <<deployment_file>>

# create a file on the fly
kubectl run redis --image=redis:alpine --dry-run=client -o yaml > redis-pod.yaml

# create a file on the fly 2: check file creation first look (whenever you're creating a pod and exposing it, it automatically creates a service, cause you can expose a service not a pod)
kubectl run httpd --image=httpd:alpine --port 80 --expose --dry-run=client -o yaml

# expose a deployment
kubectl expose deployment ingress-controller -n ingress-space --type=NodePort --port=80 --name=ingress --dry-run=client -o yaml > ingress.yaml

# get pods by by selector
kubectl get pods --selector env=dev
kubectl get pods --selector bu=finance

# get all info including replicas
kubectl get all --selector env=prod

# check service at a url (to check if your ingress rules worked) --> curl -v <<url>>
curl -v http://watch.ecom-store.com:30093/wear

# check exposed port
kubectl get pod mongo-75f59d57f4-4nd6q --template='{{(index (index .spec.containers 0).ports 0).containerPort}}{{"\n"}}'

# port forward
kubectl port-forward service/mongo 28015:27017

--- Taints ----------------------------------------------------
# taint a node
kubectl taint node node01 spray=mortein:NoSchedule

# un-taint a node (kubectl taint nodes node1 key1=value1:NoSchedule-) taint name can be found from pod description
kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule-

# label a node
kubectl label node node01 color=blue

# label all pods in a namespace
kubectl label pods --all protected=true -n sun

# create deployment with specification
kubectl create deployment blue --image=nginx --replicas=3

# get taints associated with a node
kubectl describe node node01 | grep -iF taints

--- Daemonsets ----------------------------------------------------
# get all DaemonSets in all namespaces
kubectl get DaemonSets --all-namespaces

# create a DaemonSet
# by create a deployment, remove strategy, replicas, timeouts, change kind -> deployment to daemonset
kubectl create deployment elasticsearch --image=k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml

--- Ingress --------------------------------------------------------
# create ingress
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"

# get all ingresses
kubectl get ingress --all-namespaces

# explain ingress info recursively
kubectl explain ingress --recursive | grep VERSION

--- Configs and secrets --------------------------------------------
# create a configmap
kubectl create configmap webapp-color --from-literal=APP_COLOR=darkblue --dry-run=client -o yaml > webapp-color-configmap.yaml

# create a secret
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123 --dry-run=client -o yaml > db-secret.yaml

# create tls secret
kubectl create secret tls webhook-server-tls --cert=/root/keys/webhook-server-tls.crt --key=/root/keys/webhook-server-tls.key --namespace=webhook-demo

--- Service accounts and roles --------------------------------------------
# create service called demo.
$ kubectl create sa demo

# create a role named demo that allows user to perform “get”, “watch” and “list” on pods,deploy,ds,rs,sts:
$ kubectl create role demo --verb=get,list,watch --resource=pods,deploy,ds,rs,sts

# or with all verbs
$ kubectl create clusterrole demo --verb='*' --resource=pods,deploy,ds,rs,sts

# For cluster role
$ kubectl create clusterrole demo --verb=get,list,watch --resource=pods,deploy,ds,rs,sts

# create a RoleBinding for the demo role.
$ kubectl create rolebinding demo --role=demo --user=demo

# for Cluster role
$ kubectl create rolebinding demo --clusterrole==demo --user=demo

# clusterrole binding
$ kubectl create clusterrolebinding demo-admin --clusterrole=demo --user=demo

# get created resources
$ kubectl get sa,role,rolebinding
NAME                      SECRETS   AGE
serviceaccount/default    1         2d21h
serviceaccount/demo       1         4m48s
serviceaccount/newrelic   1         3h51m

NAME                                  CREATED AT
role.rbac.authorization.k8s.io/demo   2020-12-10T19:09:01Z

NAME                                         ROLE        AGE
rolebinding.rbac.authorization.k8s.io/demo   Role/demo   21s

# validate
kubectl auth can-i create deployment --as demo # yes
kubectl auth can-i '*' ds --as demo # yes

# clean up
kubectl delete sa demo
kubectl delete role demo
kubectl delete clusterrole demo
kubectl delete rolebinding demo

--- Contexts -------------------------------------------------------
# listing contexts
kubectl config get-contexts

# switch between clusters by setting the current-context in a kubeconfig file
$ kubectl config use-context <context-name>

# set a context entry in kubeconfig
kubectl config set-context <context-name>

# if you want to change namespace preference use
kubectl config set-context <context-name> --namespace=<ns-name>

# see current context
kubectl config current-context

--- Storage Classes -----------------------------------------------
# describe a storage class
kubectl describe storageclass local-storage

# if it says noprovision that means it has no dynamic provisioning

--- Special -------------------------------------------------------
# /etc/kubernetes/manifests (if not then check the kubelet config) <-- path where static pods are located <-- create a file there with image, pod name and command
kubectl run static-busybox --restart=Never --image=busybox --dry-run=client -o yaml --command sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
ps -ef | grep /usr/bin/kubelet

# ssh to another node
ssh <<node_name>>

# drain node for maintenance
kubectl drain <node_name> --ignore-daemonsets

# drain node for maintenance forcefully
kubectl drain <node_name> --ignore-daemonsets --force

# cordon/uncordon a node (mark a node as unschedulable/schedulable)
kubectl uncordon node01

# get all configs used to create all the pods
kubectl get all --all-namespaces -o -yaml > all-config.yaml

# backup etcd
etcdctl snapshot save snapshot.db
service kube-apiserver stop
etcdctl restore snapshot.db --data-dir /var/lib/etcd-from-backup
systemctl daemon-reload
service etcd restart

# get etcd version from namespace kube-system <-- always in the namespace kube-system and master/controlplane node
kubectl describe pod etcd-controlplane -n kube-system

# address to reach etcd cluster
kubectl describe pod etcd-controlplane -n kube-system | grep -i listen-client-urls

# validate yamls
kubectl create --dry-run --validate -f <file>.yaml

# To list resources sorted by name you'll use.
kubectl get <resource> --sort-by=.metadata.name

# get taints on nodes
kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.taints}{"\n"}{end}'

# server settings
# get api server settings
kubectl describe pod kube-apiserver-controlplane -n kube-system

# os installed
cat /etc/*release*

# kubectl api-resources for shortforms/abbreviations
kubectl api-resources

# get api version
kubectl proxy 8001&
curl localhost:8001/apis/authorization.k8s.io

# alias
alias ktl='kubectl' 

--- Docker ---------------------------------------------------------
# build an image
docker build -t webapp-color .

# build a container and expose to a port
docker run -p 8282:8080 webapp-color

# check base image
docker run python:3.6 cat /etc/*release*

# tag an image
docker tag httpd fedora/httpd:version1.0

--- Kube-System-Namespace -------------------------------------------
# get all pods
kubectl get pods -n=kube-system

# get inside kube-apiserver
kubectl exec -ti kube-apiserver-controlplane -n=kube-system -- kube-apiserver -h

# check config files
grep enable-admission-plugins /etc/kubernetes/manifests/kube-apiserver.yaml
    - --enable-admission-plugins=NodeRestriction

# check admission plugins (every pod is still a linux process)
ps -ef | grep kube-apiserver | grep admission-plugins

# modify admission control
vim /etc/kubernetes/manifests/kube-apiserver.yaml

--- Helm ------------------------------------------------------------
# search helm charts for wordpress
helm search hub wordpress

# repo add
helm repo add bitnami https://charts.bitnami.com/bitnami

# search an already added repo
helm search repo joomla

# install a release from a repo/application
helm install bravo bitnami/drupal

# pull a repo in /root
helm pull --untar bitnami/apache

# after you pulled all the config is inside ./apache/values.yaml and feel free to change and save and install it by the name my-webapp
helm install my-webapp ./apache
