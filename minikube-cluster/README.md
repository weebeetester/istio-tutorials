# This document details on setting up a new minikube cluster

# 1. Setting Up Minikube Cluster

## 1.1 Starting minikube
```bash
 istio-tutorials  minikube start --vm=true -p cluster-1   // Starts minikube cluster in a VM, with cluster name cluster-1
😄  [cluster-1] minikube v1.31.2 on Darwin 13.5.2
✨  Automatically selected the hyperkit driver. Other choices: virtualbox, ssh
👍  Starting control plane node cluster-1 in cluster cluster-1
🔥  Creating hyperkit VM (CPUs=2, Memory=6000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
❗  Certificate client.crt has expired. Generating a new one...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "cluster-1" cluster and "default" namespace by default

//Note: If above step failes for some reason, just delete and recreate : https://github.com/kubernetes/minikube/issues/16503

 istio-tutorials  kubectl config current-context
cluster-1

...

 istio-tutorials  minikube profile cluster-1   //To explicitly switch to cluster-1
minikube profile was successfully set to cluster-1

>minikube version                                 //Shows minikube version
>minikube stop -p cluster-1                       //Stop the cluster
>minikube delete -p cluster-1                     //Delete the cluster
>minikube start                                   // Starts in a docker container if docker is detected, with cluser name 'minikube'
>minikube start --vm=true                         // Starts in a VM
>minikube start --memory=4096 --driver=hyperkit   //same as above

```