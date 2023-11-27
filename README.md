# k3d
k3d infr for kubernetes cluster

---
How to install k3d in your local machine.
How to create a kubernetes cluster in local machine using cli.
How to create registy locally
How to run your application on a multi cluster locally
how to administer the k3d
# To communicate between Kubectl and k8s cluster ,we need 1.kube/config file and 2. network 
--------------------------------------------
open -a docker ( If you using docker desktop)
k3d cluster list
k3d cluster start dev-cluster
kubetle cluster-info
curl -v telnet://localhost:6445 
k3d kubeconfig get dev-cluster

k get nodes
k get ns
docker ps 
k get all
k get all -n kube-system
k get all -o wide  -n kube-system
k get po -o wide -n kube-system -v=8
k api-resources  # to see the api version when write menifest files.
k apply -f my-ns.yaml
## difference between create/update and apply 
k get ns

k describe ns my-ns
# To see yaml format
k get ns my-ns -o yaml
k get all -n my-ns 
k get all
k get all --all-namespaces
k get all -A
k get pods -A
k get deployments -A
k get seervices -A
k get pods -n my-ns

kgaa

------

** A kubernetes object is a "record of intent" - once you create a the object,Kubernetes system constantly work to ensure
   that object exist.
   -- To create or modify or delete them --You need to use kubernetes API. Object like POD,ReplicaSet,DaemonSet,Deploymentn
      Services etc. u can use kubectl or python or cli to communicate with api.
      These Objects are represent something in a cluster.

** Namespaces as a virtual cluster inside your kubernetes cluster.they are all ligically isolated from each other.
   Namsespaces used for isolation.
   -- By default there are four namespaces will be created like default,kube-system,kube-public and also kube-node-lease
   -- resource quota.
** Create / Update /Delete/Read any k8s Resource /Object we can use kubectl(cli) --> Imperative approch. And Declarative approch
   ----> create a menifest files.
==============================================

K8s     API / OBJECT / RESOURCES / WORKLOADS
--------------------------------------------

POD
ReplicationController
ReplicaSet
Deployment
StatefulSet

Communication
-------------

Services :
  ClusterIp
  NodePort
  Headless
  LoadBalancer

 NetworkPolicies 

Storages :
----------

 PersistentVolume
 PersistentVolumeClaim
 StorageClass

 RBAC (Role Based Access Control)
 --------------------------------

 Role
 RoleBinding
 ClusterRole
 ClusterRoleBinding
 ServiceAccount

Scheduling and Maintence
-------------------------

Node Selector
Node Affinity
POD Affinity/AntiAffinity
Taints and Tolerations
Drain
Cardon / UnCardon

============================

A Pod always runs on a Node. smallest building block . a pod represent a running process.
- In side a pod , You can have one or more containers.

# kubectl apply -f pod.yaml --dry-run=client
# kubectl apply -f pod.yaml --dry-run=server (kubectl validate with api server)










- before you create a k3d cluster you have to start docker deamon first.

- k3d registry create my-registry
you can create a registry like this and use the registry for your images.

# You can now use the registry like this (example):
# 1. create a new cluster that uses this registry
k3d cluster create --registry-use k3d-my-registry:49552 
If you want to create a cluster by using config file like cluster.yaml/registry.yaml
# k3d cluster create -c cluster.yaml --registry-config registry.yaml
kubectl config use-context k3d-dev
kubectle cluster-info
kubectx k3d-dev
kubectl get namespaces

# ALB --> INGRESS CONTROLLER --> SERVICE ---> DEPLOYMENT(RS) --->PODS -----> CONTAINERS
# ALB / INGRESS CONTROLLER ALWAYS EXPOSE OUTSIDE OF WORLDS
# SERVICE ALWAYS EXPOSE INTERNALLY IN THE CLUSTER.

 #2. tag an existing local image to be pushed to the registry
docker tag nginx:latest k3d-my-registry:49716/mynginx:v0.1

## # Added by docker Desktop. To allow the same kube context to work on the host and the container
:
 go to vi /etc/hosts and add the line below.
127.0.0.1 kubernetes.docker.internal k3d-demo-registry


# 3. push that image to the registry
docker push k3d-my-registry:49716/mynginx:v0.1

# 4. run a pod that uses this image
kubectl run mynginx --image k3d-my-registry:49716/mynginx:v0.1

If you go to : and see there is a container running images # registry:2
docker ps -a
docker images
docker pull nginx
# docker tag the image and push it to the registry.
docker ps -f name=k3d-my-registry  (that will give detail about your registry)

Creating cluster in command line like this below

# k3d cluster create dev --registry-use k3d-my-registry:49552 --servers 2 --agents 2

when you create a cluster you will have $HOME/.kube/config file . you can talk to api server 
# cat $HOME/.kube/config | grep -i dev
kubectl get nodes
kubectl get pods
kubectl get ns
To see the traefik ,metric server info
## kubectl get deployment.apps -n kube-system

# To run the application 
kubectl apply -f deployment.yaml
# deployment -> pods -> each pod has one container in it.
# Ingress --> Service --> Deployment --> Pods --> Containers
# Docker does not know how to talk to your local machine.

k3d cluster start dev
k3d cluster start stage
k3d cluster start prod

k3d kubeconfig get dev-cluster
curl -v telnet://localhost:6445  

# To set permanent bash aliases change this file ~/.bash_aliases

k get all -n kube-system 
k get po -o wide -n kube-system
k get po -o wide -n kube-system -v=8

alias k='kubectl'
alias kc='k config view --minify | grep name'
alias kdp='kubectl describe pod'
alias c='clear'
alias kd='kubectl describe pod'
alias ke='kubectl explain'
alias kf='kubectl create -f'
alias kg='kubectl get pods --show-labels'
alias kr='kubectl replace -f'
alias ks='kubectl get namespaces'
alias l='ls -lrt'
alias kga='k get pod --all-namespaces'
alias kgaa='kubectl get all --show-labels'

=========================================

# Execute a kubectl command against all namespaces
alias kca='f(){ kubectl "$@" --all-namespaces;  unset -f f; }; f'

# Apply a YML file
alias kaf='kubectl apply -f'

# Drop into an interactive terminal on a container
alias keti='kubectl exec -ti'

# Manage configuration quickly to switch contexts between local, dev ad staging.
alias kcuc='kubectl config use-context'
alias kcsc='kubectl config set-context'
alias kcdc='kubectl config delete-context'
alias kccc='kubectl config current-context'

# List all contexts
alias kcgc='kubectl config get-contexts'

# General aliases
alias kdel='kubectl delete'
alias kdelf='kubectl delete -f'

# Pod management.
alias kgp='kubectl get pods'
alias kgpw='kgp --watch'
alias kgpwide='kgp -o wide'
alias kep='kubectl edit pods'
alias kdp='kubectl describe pods'
alias kdelp='kubectl delete pods'

# get pod by label: kgpl "app=myapp" -n myns
alias kgpl='kgp -l'

# Service management.
alias kgs='kubectl get svc'
alias kgsw='kgs --watch'
alias kgswide='kgs -o wide'
alias kes='kubectl edit svc'
alias kds='kubectl describe svc'
alias kdels='kubectl delete svc'

# Ingress management
alias kgi='kubectl get ingress'
alias kei='kubectl edit ingress'
alias kdi='kubectl describe ingress'
alias kdeli='kubectl delete ingress'

# Namespace management
alias kgns='kubectl get namespaces'
alias kens='kubectl edit namespace'
alias kdns='kubectl describe namespace'
alias kdelns='kubectl delete namespace'
alias kcn='kubectl config set-context $(kubectl config current-context) --namespace'

# ConfigMap management
alias kgcm='kubectl get configmaps'
alias kecm='kubectl edit configmap'
alias kdcm='kubectl describe configmap'
alias kdelcm='kubectl delete configmap'

# Secret management
alias kgsec='kubectl get secret'
alias kdsec='kubectl describe secret'
alias kdelsec='kubectl delete secret'

# Deployment management.
alias kgd='kubectl get deployment'
alias kgdw='kgd --watch'
alias kgdwide='kgd -o wide'
alias ked='kubectl edit deployment'
alias kdd='kubectl describe deployment'
alias kdeld='kubectl delete deployment'
alias ksd='kubectl scale deployment'
alias krsd='kubectl rollout status deployment'
kres(){
    kubectl set env $@ REFRESHED_AT=$(date +%Y%m%d%H%M%S)
}

# Rollout management.
alias kgrs='kubectl get rs'
alias krh='kubectl rollout history'
alias kru='kubectl rollout undo'

# Port forwarding
alias kpf="kubectl port-forward"

# Tools for accessing all information
alias kga='kubectl get all'
alias kgaa='kubectl get all --all-namespaces'

# Logs
alias kl='kubectl logs'
alias klf='kubectl logs -f'

# File copy
alias kcp='kubectl cp'

# Node Management
alias kgno='kubectl get nodes'
alias keno='kubectl edit node'
alias kdno='kubectl describe node'
alias kdelno='kubectl delete node'

