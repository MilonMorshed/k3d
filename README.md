# k3d
k3d infr for kubernetes cluster

---
How to install k3d in your local machine.
How to create a kubernetes cluster in local machine using cli.
How to create registy locally
How to run your application on a multi cluster locally
how to administer the k3d
# To communicate between Kubectl and k8s cluster ,we need 1.kube/config file and 2. network 

Creating cluster :
-----------------
k3d cluster create -c dev-cluster.yaml
- create a registry  # k3d registry create my-registry
- Need to find which port is runnung your registry.
    docker ps
    docker ps -f name=my-registry

k3d cluster delete dev-cluster
--------------------------------------------

commented out multiple lines   command + k + c  and uncomment  ---> command + k + u 

open -a docker ( If you using docker desktop)
k3d cluster list
k3d cluster start dev-cluster
kubectle cluster-info
curl -v telnet://localhost:6445 
k3d kubeconfig get dev-cluster

k get nodes
k describe node k3d-dev-cluster-server-0

k get ns
k get pod -o wide -n my-ns
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
k describe pod my-pod -n my-ns 
k exec -it my-pod -n my-ns -- sh 
kubectl exec POD [-c CONTAINER] [-i] [-t] [flags] [-- COMMAND [args...]]
k get pods --show labels -n my-ns
k edit svc myservice -n my-ns -o yaml
kgaa

------

** A kubernetes object is a "record of intent" - once you create a the object,Kubernetes system constantly work to ensure that object exist.
   -- To create or modify or delete them --You need to use kubernetes API. Object like POD,ReplicaSet,DaemonSet,Deploymentn
      Services etc. u can use kubectl or python or cli to communicate with api.
      These Objects are represent something in a cluster.
------
** Namespaces as a virtual cluster inside your kubernetes cluster.they are all logically isolated from each other.
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
 -------------------------------------
 

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
Service :
---------

  - A service is make our pods discoverable inside the cluster Or exposing them to the internet
  - When we create a service -
      we get one virtual IP (Cluster IP) and it will get registered to the DNS(kube-dns).
      After registering , other Pods can find and talk to the service by using service name.
  - Service is a logical concept, real work done by kube-proxy on each node.
  - Redirect Virtual IP to Pod IP.
  - labels and selector are very imporant in creating svc . 
  - labels and selector has to be same.
    k get pods --show-labels -n my-ns
  - You can modify the labels in pod.yaml or you can modify the selector in service.yaml
  - To edit the yaml file you can also do that by imperative approch. like 

  k edit svc nginxsvc -n my-ns   
  k get ep -n my-ns

  ------------
  FQDN ? --> Fully Qualified Domain Name .When do You have to use FQDN?
  <serviceName>.<namespaceName>.svc.cluster.local
  nginxsvc.my-ns.svc.cluster.local

  curl -v testsvc.default.svc.cluster.local:8080
  ---------
  - cluster capacity ?
  - resources quota --> 
        resources:
          request:
             cpu: 100m
             memory: 200mi ;kubelet will check and Sum all the existing containers resources request
             < Availalbe resources in the node. if not available pod get to pending state.

   - resource quota : Resource quota is just a limit how much resources can you use in a namespace.
   - resource request : you can do it in Pod level.
   20 Nodes , Each node 128 GB Ram,
                          64 core cpu.
   -------------------------------------------------------------

   Pod Lifecycle
   -------------
   1. Make a pod request to API Server using local pod manifest file
   2. The API server saves the info for the pod in ETCD.
   3. The Scheduler finds the unscheduled pod and schedules it to node.
   4. Kubelet running on the node,see  the pod scheduled and fires up docker.
   5. Docker runs the container.
   6. The entire lifecyle state of the pod is stored in ETCD.

   -------
   - Pod is ephemeral(last for very short time) and won't be rescheduled to a new node once it dies.
     that's why you shouldn't directly create a pod instead of using controller like deployment,replicaSets
     etc.
     Pod should be created and managed by some controllers.

     - ReplicationController
     - ReplicaSet
     - DeamonSet
     - Deployments
     - StatefulSet

   Static Pods:
   -----------
    - Static pod are managed by the kubelet and API server does not have any control over the pods. The static pods running 
      on a node are visible on the API server.There is no health check for the static pods.
      kubelet is responsible to watch each static pod and restart the pod if it's get crashed.

      kubectl scale rc <replicationControllerName> --replicas 3 -n my-ns -----> imperative approch.
      kubectl run testpod --image=nginx --labels="app=nginx" --port=80 -n default --dry-run=client
      kubectl run testpod --image=nginx --labels="app=nginx" --port=80 -n default --dry-run=client -o yaml
      kubectl run testpod --image=nginx --labels="app=nginx" --port=80 -n default --dry-run=client -o yaml > testpod.yaml

      ReplicaSet: ReplicaSet is next generation.ReplicaSet also create and manage pods and we can scale out the pods. 
      ----
                  Only defferance b/w rc and rs is in Selecter Support.and version is different v1. apps/v1
                   - RC Supports only Equality Based Selector .Selector is not a mandatory.and 
                     selector:
                       app: nodeapp # Equality based 
                   - RS Supports both Equality and Set (Expression )Based Selector.Selector is mandatory.
                     selector:
                       matchLabels:
                         app: nodeapp # Equality based
                     -------
                     selector:
                       matchExpression:  # set based 
                       - key: app
                         operator: In (operator)
                         values:
                         - "nodeapp"
                         - "node" 
                    template # and then specification for PODS
      Still you can create a pod, delete manage the pod using RS/RC.

  ---------
  DeamonSet(DS):
  -------------
  RC / RS are the replica mode. You cann't scale your container directly.
  A DaemonSet make sure that all or some kubernetes Nodes run a copy of a pod.
  When a new node is added to the cluster , a Pod is added to it to mactch the rest of the node is removed from the cluster,
  the pod is garbage collected.

  DeamonSet is global mode like docker swarm. It is exactly like ReplicaSet only .Only difference is that it's not allow to have 
  replicas fields in a manifest files. 

  DaemonSet is used for agent based application like monitoring ,login agent ,logstash etc
      
  -----
  taint:  <key> = <value>: <Effect>
          node-role.kubernetes.io/master:NoSchedule

          Effect has three option 1. NoSchedule 2. PreferNoSchedule 3. NoExecute 
          toleration:
          - key: node-role.kubernetes.io/master
            operator: Exists or equal
            effect: NoSchedule
  toleration: 

          You have to configure in pod spec lavel ;
          tolerations: 
          - key:
            value:
            effect:
          containers:  

    -------

    Deployments:
    ------------
     - Deployments is the recommended way to deploy Pod or rs
     Key benefits of using Deployments:
     -----------------------------------

     - Deploy RS
     - Update pods (PodTemplateSpec) zero downtime.
     - Rolling to older Deployment version.
     - Scale Deployment up or down.
     - Pause and resume the deployment.
     - Clean up older RS that you don't need any more.

     -------

     kubectl scale deployment [DEPLOYMENT_NAME] --replica[NUMBER_OF_REPLICAS]
     k rollout status deployment nginx-deploment
     k get deployment
     k set image deployment nginx0deployment nginx-container=nginx:latest --record
     k get replicaset
     k rollout history deployment nginx-deployment
     k rollout history deployment nginx-deployment --revision=1
     k rollout undo deployment nginx-deployment --to-revision=1

     Deployment create --> RS ---> RS will manage the pods
     Deployment has two strategies: 
            1. Recreate (when update it will terminate old pod and recreate the new pods.there will be a downtime)
            2. Rolling Deployment(default deployment to k8s.It works very slow,one by one, There won't be any downtime)
            First create new pods --> delete one old pod ---> give break for 30sec---> then create another.

      HPA AND METRIC SERVER:
      ---------------------
      - HPA depends on metrics .HPA will communicate with the metric servier API and to  collect the info about
      the CPU ,MEMORY utiliztion of POD .(has to install metrics server).
      - Metric server will communicate with kubelet. Kubelet will provide the info to metric server.
      - What is heapster and what is metric server? they both collect the values of resources. heapster 
        is deprecated old version. Metric server is recommended . 

      - kubectl top nodes
      - kubectl top pods
      - kubectl top pods -A --containers=true

      ** resource request and limits when you create a containers.
      - Resources Request: Minimum resources should be available.
      - Resources Limits: Max resources can be used.
      ** OOMkilled (out of memory killed).

      ** Resource limits should be always greater or equal (>=)  than Resource Request.
      Resources -----> Request and Limits
       
      ----
      Deployment: It's recommend for State less Application. still you can make your deployment statefulSet
      ----------
                  by adding volume .
      StatefulSet:
      ------------
       This is designed for deploying and managing Stateful Application like databases, 
       jenkins,nexus,kafka,cluster etc.
       - If you want to create StatefulSet in the same cluster . You need to svc type cluster IP.

      -------------
      volume: 
      -------



   - Pending (because it's not scheduled , Insufficent resources or some other cloud)
   troubleshoot: kubectl describe pod <podName>.
   - node had a disk presure ( That means that node is using too much disk space or is using disk space too fast. Automatically 
     it will solve this .That is important to monitor because you  might to add more disk spaces )
   - Node does not have a sufficient memory

   - kubectl k3d exec -it NODE_NAME /bin/sh
   - k k3d exec
   - CrashLoopBackOff (Some reason your container process is not starting successfully because of some code 
                       issue,configuration issue,application,images etc) contact with developer.
                       may be correct the code or recreate the code or images.
    troubleshoot: kubectl describe pod <podName>
                  kubectl get pod -n ns 
                  kubectl logs <podName> -c <containerName> 

   - ImagePullBackOff / ErrorImagePull (Not able to pull the images. There is two reasons
                  1.Wrong registry details ,if it is in private registry 2.Authentication problem)
   - Running
   - CreateContainerConfigError ( configMap ,secrete .You are reffering the configMap and secret in your 
                  pod manifest file but you didn't create those or don't have those file in your cluster.)
   - If the nodes are ready . We can check like 
         - kubectl get nodes
         - kubectl describe node <nodeName>. to see is there any memory or disk presures


    -------------------------
       
         
A Pod always runs on a Node. smallest building block . a pod represent a running process.
- Inside a pod , You can have one or more containers.

# kubectl apply -f pod.yaml --dry-run=client
# kubectl apply -f pod.yaml --dry-run=server (kubectl validate with api server)


----------------
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

