# Koinos on Kubernetes

The Koinos Blockchain consists of multiple interdependent microservices that communicate via AMQP. Each microservice has been dockerized and can easily be run as a unit with Docker Compose.

Images are automatically uploaded to [Docker Hub](https://hub.docker.com/u/koinos). `latest` tracks `master` on each microservice repo. Feature branches are uploaded with their branch name as the image tag.

The dockerized microservices can be hosted to Kubernetes. This repository will help you to set up your own Koinos Kubernetes cluster.



## Kubernetes
New to Kubernetes? Get familiarized with Kubernetes here: [The Definitive Guide to Kubernetes](https://gabrieltanner.org/blog/the-definitive-guide-to-kubernetes)

### Persistent Volume Claims (PVC)
Kubernetes works with Persistent Volumes for mounting data the microservices can use. It's the like the base directory `~/.koinos` on the host when using `docker-compose`.

You can update the size of the volume inside `koinos-data-persistentvolumeclaim.yaml`
```yaml
spec:
  resources:
    requests:
      storage: 10Gi
```

**More info:**
- [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [Kubernetes Persistent Volumes: Examples & Best Practices](https://loft.sh/blog/kubernetes-persistent-volumes-examples-and-best-practices/)
- [Kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)


## Prerequisites

### Install Docker with Kubernetes
To set up a cluster on your machine you will need to install Docker on MacOS or Windows first . You can follow their instructions for installation [here](https://www.docker.com/products/docker-desktop).

After installing Docker you will have to [enable Kubernetes](https://docs.docker.com/desktop/kubernetes/#:~:text=To%20enable%20Kubernetes%20in%20Docker,then%20click%20Install%20to%20confirm) inside the Docker settings.

### Kubectl
You can also only install [kubectl](https://kubernetes.io/docs/tasks/tools/) if you don't want to run a cluster locally.

### Setup cluster at cloud provider like DigitalOcean
...

## Config node

The config service is responsible for setting up some configurations required by the Koinos microservices. This service needs to run only once (unless you are updating the configuration) and can be deleted afterwards.
We will need to build a custom Docker image for our configuration so the service can be deployed.

It will create the following on the PVC:
- Create a new private key
- Add the genesis_data
- Copy your configuration to the persistence volume

### Build image
We will use the default configuration inside `.\config\default-config.yml` for our cluster including the `block_producer`, `jsonrpc` and `transaction_store` microservices.

First create an account on [Docker Hub](https://hub.docker.com/) and login using your terminal.
```shell
docker login

# Build + tag the Docker image
docker build ./config -t {username}/{repository}:{versionTag}

# Like:
docker build ./config -t me/koinos-config:v0.1.0

# Push the image to Docker Hub
docker push {username}/{repository}:{versionTag}
```

#### Caveat for Windows users
Windows users might experience some issues with line endings when running the built config image. Make sure the `run.sh` file line endings are set to `LF` instead of `CRLF` before building the image.

Otherwise, you could run into the following error:
```shell
/koinos-config/run.sh: line 2: $'\r': command not found
: invalid optionun.sh: line 3: set: -
set: usage: set [-abefhkmnptuvxBCHP] [-o option-name] [--] [arg ...]
: invalid optionun.sh: line 4: set: -
set: usage: set [-abefhkmnptuvxBCHP] [-o option-name] [--] [arg ...]
/koinos-config/run.sh: line 5: $'\r': command not found
/koinos-config/run.sh: line 22: syntax error: unexpected end of file
```

### Deployment file
Update the image in the deployment file `.\k8\1-node\config-deployment.yaml` with the created image.
```yaml
    spec:
      containers:
        - image: {username}/{repository}:{versionTag}
```



## 1. Run Koinos with Kubernetes

Kubernetes works with contexts for accessing multiple clusters.

```shell
# List Kubernetes contexts
kubectl config get-contexts

CURRENT   NAME               CLUSTER            AUTHINFO                 NAMESPACE
*         do-ams3-koinos-1   do-ams3-koinos-1   do-ams3-koinos-1-admin
          docker-desktop     docker-desktop     docker-desktop

# Use the local Docker context
kubectl config use-context docker-desktop

# Use the context on DigitalOcean
kubectl config use-context do-ams3-koinos-1
```

### Setup cluster
```shell
# Apply the namespace configuration first, before the other k8s configurations
kubectl apply -f .\k8s\1-node\koinos-namespace.yaml,.\k8s\1-node

# When config pod has run successfully you can delete it
kubectl delete -n default deploy config

# You could also manually specify which configurations you want to deploy. 
# For example when disabling specific services. 
kubectl apply -f .\k8s\1-node\koiner-namespace.yaml,.\k8s\1-node\koinos-data-persistentvolumeclaim.yaml,.\k8s\1-node\config-deployment.yaml,.\k8s\1-node\amqp-service.yaml,.\k8s\1-node\jsonrpc-tcp-service.yaml,.\k8s\1-node\p2p-service.yaml,.\k8s\1-node\amqp-deployment.yaml,.\k8s\1-node\block-producer-deployment.yaml,.\k8s\1-node\block-store-deployment.yaml,.\k8s\1-node\chain-deployment.yaml,.\k8s\1-node\contract-meta-store-deployment.yaml,.\k8s\1-node\jsonrpc-deployment.yaml,.\k8s\1-node\mempool-deployment.yaml,.\k8s\1-node\p2p-deployment.yaml,.\k8s\1-node\transaction-store-deployment.yaml
```

### Delete deployments
```
kubectl delete -n koinos deploy,svc,pvc --all
kubectl delete namespace koinos
```

### Retrieve private key
...

## 2. Add Ingress

More info:
- [1](https://kubernetes.github.io/ingress-nginx/)
- [2](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)
- [3](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [Understanding Kubernetes LoadBalancer vs NodePort vs Ingress](https://platform9.com/blog/understanding-kubernetes-loadbalancer-vs-nodeport-vs-ingress/)

### Install nginx
https://kubernetes.github.io/ingress-nginx/deploy/#quick-start
#### Install nginx ingress local:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml

#### Install nginx ingress on DO:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/do/deploy.yaml

### Apply ingress
```
...
```



## 3. Add custom service


## Next steps

### Scaling
- [Architecting Kubernetes clusters — how many should you have?](https://learnk8s.io/how-many-clusters)
- [Architecting Kubernetes clusters — choosing a worker node size](https://learnk8s.io/kubernetes-node-size)

### Security
- ...
