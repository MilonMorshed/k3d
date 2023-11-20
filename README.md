# k3d
k3d infr for kubernetes cluster

---
How to install k3d in your local machine.
How to create a kubernetes cluster in local machine using cli.
How to create registy locally
How to run your application on a multi cluster locally
how to administer the k3d

- before you create a k3d cluster you have to start docker deamon first.

- k3d registry create demo-registry
you can create a registry like this and use the registry for your images.

1. create a new cluster that uses this registry.
k3d cluster create --registry-use k3d-demo-registry:54793

2. tag an existing local image to be pushed to the registry.
docker tag nginx:latest k3d-demo-registry:54793/mynginx:v0.1

3. push that image to the registry
docker push k3d-demo-registry:54793/mynginx:v0.1

4. run a pod that uses this image
kubectl run mynginx --image k3d-demo-registry:54793/mynginx:v0.1

