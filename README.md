# k3d
k3d infr for kubernetes cluster

---
How to install k3d in your local machine.
How to create a kubernetes cluster in local machine using cli.
How to create registy locally
How to run your application on a multi cluster locally
how to administer the k3d

- before you create a k3d cluster you have to start docker deamon first.

- k3d registry create my-registry
you can create a registry like this and use the registry for your images.

# You can now use the registry like this (example):
# 1. create a new cluster that uses this registry
k3d cluster create --registry-use k3d-my-registry:49552 
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
# To see the traefik ,metric server info
kubectl get deployment.apps -n kube-system
# To run the application 
kubectl apply -f deployment.yaml
# deployment -> pods -> each pod has one container in it.
# Ingress --> Seervice --> Deployment --> Pods --> Containers
# Docker does not know how to talk to your local machine.


