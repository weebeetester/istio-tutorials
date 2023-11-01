# Overview

Purpose of this document is to help learn istio and it's associated concepts, with hands-on approach. Information on this page comes from various sources including official istio documentation. 

---

# 1. Setting Up Minikube Cluster

## 1.1 Starting minikube
```bash
 > minikube start --vm=true -p cluster-1   // Starts minikube cluster in a VM, with cluster name cluster-1
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

 >kubectl config current-context
cluster-1

...

 > minikube profile cluster-1   //To explicitly switch to cluster-1
minikube profile was successfully set to cluster-1

>minikube version                                 //Shows minikube version
>minikube stop -p cluster-1                       //Stop the cluster
>minikube delete -p cluster-1                     //Delete the cluster
>minikube start                                   // Starts in a docker container if docker is detected, with cluser name 'minikube'
>minikube start --vm=true                         // Starts in a VM
>minikube start --memory=4096 --driver=hyperkit   //same as above
```
---

# 2. Istio Installation & Setup
## 2.1 Prerequisites: Make sure that cluster is up and running
 >minikube start --vm=true -p cluster-1

 >minikube profile cluster-1 

## 2.2 Enable ingress addons for Istio
```bash
>minikube addons enable ingress -p cluster-1  
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.8.1
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```


## 2.3 Installing Istio
You can install istio by multiple ways, as described at https://istio.io/latest/docs/setup/install/. 
For our learning purpose, we will use istioctl.
Start with downloading istio latest release
```bash
// You may decide to download this at istio-tutorials/istio
istio-tutorials/istio > curl -L https://git.io/getLatestIstio | sh -
curl -L https://git.io/getLatestIstio | sh -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100  4899  100  4899    0     0   6112      0 --:--:-- --:--:-- --:--:-- 32443

Downloading istio-1.19.3 from https://github.com/istio/istio/releases/download/1.19.3/istio-1.19.3-osx.tar.gz ...
Istio 1.19.3 Download Complete!

Istio has been successfully downloaded into the istio-1.19.3 folder on your system.

Next Steps:
See https://istio.io/latest/docs/setup/install/ to add Istio to your Kubernetes cluster.

To configure the istioctl client tool for your workstation,
add the /Users/puneet/workspace/repositories/istio/bin/istio-1.19.3/bin directory to your environment path variable with:
         export PATH="$PATH:/Users/puneet/workspace/repositories/istio/bin/istio-1.19.3/bin"

Begin the Istio pre-installation check by running:
         istioctl x precheck 

Need more information? Visit https://istio.io/latest/docs/setup/install/ 
```



## 2.4 Make istioctl accessible
```bash
 istio-tutorials  export PATH=$PWD/istio/istio-1.19.3/bin:$PATH
 istio-tutorials  istioctl version
no ready Istio pods in "istio-system"
1.19.3
 istio-tutorials  
```

## 2.5 Lets check if istio is installed on our cluster(it is not)
```bash
 istio-tutorials  istioctl verify-install
0 Istio control planes detected, checking --revision "default" only
error while fetching revision : the server could not find the requested resource
0 Istio injectors detected
Error: could not load IstioOperator from cluster: the server could not find the requested resource. Use --filename
 
 // Notice that there is no istio-system namespace yet
 istio-tutorials  kubectl get ns -A                                                                                                                                                                                                 ✘
NAME              STATUS   AGE
default           Active   7m52s
ingress-nginx     Active   4m57s
kube-node-lease   Active   7m52s
kube-public       Active   7m52s
kube-system       Active   7m52s
```



## 2.6 Lets install istio in our cluster now 

### We will use demo profile
```bash
 istio-tutorials  istioctl install --set profile=demo -y
✔ Istio core installed                                                                                                                                                                                        
✔ Istiod installed                                                                                                                                                                                            
✔ Egress gateways installed                                                                                                                                                                                   
✔ Ingress gateways installed                                                                                                                                                                                  
✔ Installation complete                                                                                                                                                                                       Made Made this installation the default for injection and validation.
 istio-tutorials  
```

## 2.7 Verify installation
### Once installed, use verify-install command to check if everything was installed without any problems
```bash
 istio-tutorials  istioctl verify-install
1 Istio control planes detected, checking --revision "default" only
✔ Deployment: istio-ingressgateway.istio-system checked successfully
✔ PodDisruptionBudget: istio-ingressgateway.istio-system checked successfully
✔ Role: istio-ingressgateway-sds.istio-system checked successfully
✔ RoleBinding: istio-ingressgateway-sds.istio-system checked successfully
✔ Service: istio-ingressgateway.istio-system checked successfully
✔ ServiceAccount: istio-ingressgateway-service-account.istio-system checked successfully
✔ Deployment: istio-egressgateway.istio-system checked successfully
✔ PodDisruptionBudget: istio-egressgateway.istio-system checked successfully
✔ Role: istio-egressgateway-sds.istio-system checked successfully
✔ RoleBinding: istio-egressgateway-sds.istio-system checked successfully
✔ Service: istio-egressgateway.istio-system checked successfully
✔ ServiceAccount: istio-egressgateway-service-account.istio-system checked successfully
✔ ServiceAccount: istio-reader-service-account.istio-system checked successfully
✔ CustomResourceDefinition: wasmplugins.extensions.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: destinationrules.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: envoyfilters.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: gateways.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: proxyconfigs.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: serviceentries.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: sidecars.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: virtualservices.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: workloadentries.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: workloadgroups.networking.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: authorizationpolicies.security.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: peerauthentications.security.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: requestauthentications.security.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: telemetries.telemetry.istio.io.istio-system checked successfully
✔ CustomResourceDefinition: istiooperators.install.istio.io.istio-system checked successfully
✔ ClusterRole: istiod-clusterrole-istio-system.istio-system checked successfully
✔ ClusterRole: istiod-gateway-controller-istio-system.istio-system checked successfully
✔ ClusterRoleBinding: istiod-clusterrole-istio-system.istio-system checked successfully
✔ ClusterRoleBinding: istiod-gateway-controller-istio-system.istio-system checked successfully
✔ ConfigMap: istio.istio-system checked successfully
✔ Deployment: istiod.istio-system checked successfully
✔ ConfigMap: istio-sidecar-injector.istio-system checked successfully
✔ MutatingWebhookConfiguration: istio-sidecar-injector.istio-system checked successfully
✔ PodDisruptionBudget: istiod.istio-system checked successfully
✔ ClusterRole: istio-reader-clusterrole-istio-system.istio-system checked successfully
✔ ClusterRoleBinding: istio-reader-clusterrole-istio-system.istio-system checked successfully
✔ Role: istiod.istio-system checked successfully
✔ RoleBinding: istiod.istio-system checked successfully
✔ Service: istiod.istio-system checked successfully
✔ ServiceAccount: istiod.istio-system checked successfully
✔ ValidatingWebhookConfiguration: istio-validator-istio-system.istio-system checked successfully
Checked 15 custom resource definitions
Checked 3 Istio Deployments
✔ Istio is installed and verified successfully
 istio-tutorials  

 
 // You can see there is istio-system namespace avaialble now
 istio-tutorials  kubectl get ns -A
NAME              STATUS   AGE
default           Active   13m
ingress-nginx     Active   10m
istio-system      Active   3m28s
kube-node-lease   Active   13m
kube-public       Active   13m
kube-system       Active   13m
 istio-tutorials 
```

## 2.8 Even if istio is installed, it has not been enabled yet

### Now, if we deploy any workload at this moment, well, you won't see any sidecar yet.
```bash
 istio-tutorials  kubectl apply -f istio/istio-1.19.3/samples/bookinfo/platform/kube/bookinfo.yaml 
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created

istio-tutorials  kubectl get pods   //shows only one container per pod
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-5f4d584748-4b7v6      1/1     Running   0          2m1s
productpage-v1-564d4686f-r9xhn   1/1     Running   0          2m1s
ratings-v1-686ccfb5d8-9n8z5      1/1     Running   0          2m1s
reviews-v1-86896b7648-scgkf      1/1     Running   0          2m1s
reviews-v2-b7dcd98fb-jz7h5       1/1     Running   0          2m1s
reviews-v3-5c5cc7b6d-qmt9l       1/1     Running   0          2m1s
```

## 2.9 Enable Istio
###  In order for proxy/envoy sidecar to be injected, the namespace has to be applied with a label
```bash
 istio-tutorials  istioctl analyze
Info [IST0102] (Namespace default) The namespace is not enabled for Istio injection. Run 'kubectl label namespace default istio-injection=enabled' to enable it, or 'kubectl label namespace default istio-injection=disabled' to explicitly mark it as not needing injection.
```


### So we must label the namespace (in which the workloads are being deployed)
```bash
// first delete the workload you just deployed
 istio-tutorials  kubectl delete -f istio/istio-1.19.3/samples/bookinfo/platform/kube/bookinfo.yaml
service "details" deleted
serviceaccount "bookinfo-details" deleted
deployment.apps "details-v1" deleted
service "ratings" deleted
serviceaccount "bookinfo-ratings" deleted
deployment.apps "ratings-v1" deleted
service "reviews" deleted
serviceaccount "bookinfo-reviews" deleted
deployment.apps "reviews-v1" deleted
deployment.apps "reviews-v2" deleted
deployment.apps "reviews-v3" deleted
service "productpage" deleted
serviceaccount "bookinfo-productpage" deleted
deployment.apps "productpage-v1" deleted


// Now, label the namespace
 istio-tutorials  kubectl label namespace default istio-injection=enabled
namespace/default labeled

// You can check the status again, should have no validation issue
 istio-tutorials  istioctl analyze

✔ No validation issues found when analyzing namespace: default.

// Now, redeploy the workload ( you should see additional container per pod )
 istio-tutorials  kubectl apply -f istio/istio-1.19.3/samples/bookinfo/platform/kube/bookinfo.yaml 
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
 
// You will see 2 containers per pod now
 istio-tutorials  kubectl get pods                                                           
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-5f4d584748-ljchp      2/2     Running   0          38s
productpage-v1-564d4686f-5dbkt   2/2     Running   0          38s
ratings-v1-686ccfb5d8-snfsz      2/2     Running   0          38s
reviews-v1-86896b7648-ztr2g      2/2     Running   0          38s
reviews-v2-b7dcd98fb-ms4t8       2/2     Running   0          38s
reviews-v3-5c5cc7b6d-l527n       2/2     Running   0          38s
```
---

# 3. Install Kiali (used for visualizing+managing service mesh)
## 3.1 Install kiali using addons
### This will install many addons including kiali, jaeger, prometheus, grafana and others
```bash
 istio-tutorials  kubectl apply -f istio/istio-1.19.3/samples/addons 
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
serviceaccount/loki created
configmap/loki created
configmap/loki-runtime created
service/loki-memberlist created
service/loki-headless created
service/loki created
statefulset.apps/loki created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
 istio-tutorials  

// Lets check if kiali is deployed in istio-system ns
 istio-tutorials  kubectl -n istio-system rollout status deployment kiali 
deployment "kiali" successfully rolled out

 istio-tutorials  kubectl -n istio-system get service kiali
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
kiali   ClusterIP   10.97.228.174   <none>        20001/TCP,9090/TCP   118s
 
// Lets start kiali dashboard
 istio-tutorials  istioctl dashboard kiali
http://localhost:20001/kiali
// Above will open browser with http://localhost:20001/kiali/console/services?duration=60&refresh=10000&namespaces=default
```
---

# 4. Create Traffic into your mesh
## 4.1 First, export IP & Port
```bash
 istio-tutorials  export INGRESS_HOST=$(minikube ip -p cluster-1)
 istio-tutorials  echo $INGRESS_HOST

 istio-tutorials  export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
 istio-tutorials  echo $INGRESS_PORT
```
## 4.2 Check if accessible from outside cluster
### Now, if you try to test using curl from outside cluster, it will fail as there is no ingress created yet
```bash
istio-tutorials  echo http://$INGRESS_HOST:$INGRESS_PORT/productpage
http://192.168.66.3:31483/productpage

istio-tutorials  curl "http://$INGRESS_HOST:$INGRESS_PORT/productpage"                                                                                                   ✘
curl: (7) Failed to connect to 192.168.66.3 port 31483 after 108 ms: Couldn't connect to server
```
## 4.3 Create a Gateway
### Lets create a gateway(which will configure service mesh to accept traffic from outside the cluster). There's also a virtual service in below file. 

```bash
 istio-tutorials  cat istio-basics/workspace/traffic-management/ingress/bookinfo-gateway.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 8080
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
 istio-tutorials  kubectl apply -f  istio-basics/workspace/traffic-management/ingress/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created

## Gateway
// Gateway Captures the traffic comming via Ingress-Gateway-Controller, on specific host/port. But the traffic then remains at Gateway with no where to go ( thats where Virtual-service comes into play).

## Virtual Service (when used with Gateway)
Virtual Service (when used with Gateway) routes the traffic associated with specific gateways/hosts, and mathcing rules to a destination (a kubernetes service on specific port)
A virtual service lets you configure how requests are routed to a service within an Istio service mesh, building on the basic connectivity and discovery provided by Istio and your platform.
https://istio.io/latest/docs/concepts/traffic-management/#introducing-istio-traffic-management


// You can check created gw and vs (just like any other kubernetes object)
 istio-tutorials  kubectl get gw,vs        
NAME                                           AGE
gateway.networking.istio.io/bookinfo-gateway   46s

NAME                                          GATEWAYS               HOSTS   AGE
virtualservice.networking.istio.io/bookinfo   ["bookinfo-gateway"]   ["*"]   46s

```
## 4.4 Now if you try above curl again (or in browser), you should get a response
```bash
istio-tutorials  curl  http://$INGRESS_HOST:$INGRESS_PORT/productpage 
<!DOCTYPE html>
<html>...

istio-tutorials  echo  http://$INGRESS_HOST:$INGRESS_PORT/productpage 
http://192.168.72.3:30217/productpage (you can try this in browser)
```

## 4.5 To generate traffic, you can use below command to call curl in loop, sending output to /dev/null
```bash
istio-tutorials  while sleep 0.01;do curl -sS 'http://'"$INGRESS_HOST"':'"$INGRESS_PORT"'/productpage' &> /dev/null ; done
```

## 4.6 Open Kiali dashboard (in other terminal) 
```bash
istio-tutorials  export PATH=$PWD/istio/istio-1.19.3/bin:$PATH
istio-tutorials  istioctl dashboard kiali
http://localhost:20001/kiali
```
---

# 5. Explore Istio objects
## 5.1 All deployed istio object  within istio-system namespace
```bash
 istio-tutorials  kubectl -n istio-system get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/grafana-5f9b8c6c5d-2mlhw               1/1     Running   0          48m
pod/istio-egressgateway-556f6f58f4-6fnlj   1/1     Running   0          60m
pod/istio-ingressgateway-9c8b9b586-fb4wk   1/1     Running   0          60m
pod/istiod-644f5d55fc-85jq5                1/1     Running   0          60m
pod/jaeger-db6bdfcb4-pglxp                 1/1     Running   0          48m
pod/kiali-7c9d5f9f96-hmcpb                 1/1     Running   0          48m
pod/prometheus-5d5d6d6fc-pltrx             2/2     Running   0          48m

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
service/grafana                ClusterIP      10.107.64.30     <none>        3000/TCP                                                                     48m
service/istio-egressgateway    ClusterIP      10.98.152.108    <none>        80/TCP,443/TCP                                                               60m
service/istio-ingressgateway   LoadBalancer   10.109.74.203    <pending>     15021:30413/TCP,80:30217/TCP,443:32101/TCP,31400:30739/TCP,15443:32044/TCP   60m
service/istiod                 ClusterIP      10.108.228.227   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        60m
service/jaeger-collector       ClusterIP      10.110.120.249   <none>        14268/TCP,14250/TCP,9411/TCP,4317/TCP,4318/TCP                               48m
service/kiali                  ClusterIP      10.97.228.174    <none>        20001/TCP,9090/TCP                                                           48m
service/loki-headless          ClusterIP      None             <none>        3100/TCP                                                                     48m
service/prometheus             ClusterIP      10.99.97.43      <none>        9090/TCP                                                                     48m
service/tracing                ClusterIP      10.108.173.233   <none>        80/TCP,16685/TCP                                                             48m
service/zipkin                 ClusterIP      10.103.36.69     <none>        9411/TCP                                                                     48m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana                1/1     1            1           48m
deployment.apps/istio-egressgateway    1/1     1            1           60m
deployment.apps/istio-ingressgateway   1/1     1            1           60m
deployment.apps/istiod                 1/1     1            1           60m
deployment.apps/jaeger                 1/1     1            1           48m
deployment.apps/kiali                  1/1     1            1           48m
deployment.apps/prometheus             1/1     1            1           48m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-5f9b8c6c5d               1         1         1       48m
replicaset.apps/istio-egressgateway-556f6f58f4   1         1         1       60m
replicaset.apps/istio-ingressgateway-9c8b9b586   1         1         1       60m
replicaset.apps/istiod-644f5d55fc                1         1         1       60m
replicaset.apps/jaeger-db6bdfcb4                 1         1         1       48m
replicaset.apps/kiali-7c9d5f9f96                 1         1         1       48m
replicaset.apps/prometheus-5d5d6d6fc             1         1         1       48m
```

## 5.2 View service istio-ingressgateway 
### This is the service in front of actual gateway controller (the pod using envoy proxy)
```bash
 istio-tutorials  kubectl -n istio-system get svc istio-ingressgateway -o yaml                                                                                                                                            ✘
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: "2023-10-29T16:21:31Z"
  labels:
    app: istio-ingressgateway
    install.operator.istio.io/owning-resource: installed-state
    install.operator.istio.io/owning-resource-namespace: istio-system
    istio: ingressgateway
    istio.io/rev: default
    operator.istio.io/component: IngressGateways
    operator.istio.io/managed: Reconcile
    operator.istio.io/version: 1.19.3
    release: istio
  name: istio-ingressgateway
  namespace: istio-system
  resourceVersion: "1206"
  uid: 1d3c48e7-f0a9-4158-8eb0-73d92160fb8c
spec:
  allocateLoadBalancerNodePorts: true
  clusterIP: 10.109.74.203
  clusterIPs:
  - 10.109.74.203
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: status-port
    nodePort: 30413
    port: 15021
    protocol: TCP
    targetPort: 15021
  - name: http2
    nodePort: 30217
    port: 80
    protocol: TCP
    targetPort: 8080
  - name: https
    nodePort: 32101
    port: 443
    protocol: TCP
    targetPort: 8443
  - name: tcp
    nodePort: 30739
    port: 31400
    protocol: TCP
    targetPort: 31400
  - name: tls
    nodePort: 32044
    port: 15443
    protocol: TCP
    targetPort: 15443
  selector:
    app: istio-ingressgateway
    istio: ingressgateway
  sessionAffinity: None
  type: LoadBalancer
status:
  loadBalancer: {}
 ```
## 5.3 View actual gateway controller pod
```bash
 istio-tutorials  kubectl -n istio-system get pod istio-ingressgateway-9c8b9b586-fb4wk -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    istio.io/rev: default
    prometheus.io/path: /stats/prometheus
    prometheus.io/port: "15020"
    prometheus.io/scrape: "true"
    sidecar.istio.io/inject: "false"
  creationTimestamp: "2023-10-29T16:21:31Z"
  generateName: istio-ingressgateway-9c8b9b586-
  labels:
    app: istio-ingressgateway
    chart: gateways
    heritage: Tiller
    install.operator.istio.io/owning-resource: unknown
    istio: ingressgateway
    istio.io/rev: default
    operator.istio.io/component: IngressGateways
    pod-template-hash: 9c8b9b586
    release: istio
    service.istio.io/canonical-name: istio-ingressgateway
    service.istio.io/canonical-revision: latest
    sidecar.istio.io/inject: "false"
  name: istio-ingressgateway-9c8b9b586-fb4wk
  namespace: istio-system
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: istio-ingressgateway-9c8b9b586
    uid: 6b012b42-1f81-4a94-b2a3-5039126937b5
  resourceVersion: "1239"
  uid: a7fa9fc6-cf9b-41ac-a49d-2dfb55a0326a
spec:
  affinity:
    nodeAffinity: {}
  containers:
  - args:
    - proxy
    - router
    - --domain
    - $(POD_NAMESPACE).svc.cluster.local
    - --proxyLogLevel=warning
    - --proxyComponentLogLevel=misc:error
    - --log_output_level=default:info
    env:
    - name: JWT_POLICY
      value: third-party-jwt
    - name: PILOT_CERT_PROVIDER
      value: istiod
    - name: CA_ADDR
      value: istiod.istio-system.svc:15012
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: spec.nodeName
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.namespace
    - name: INSTANCE_IP
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: status.podIP
    - name: HOST_IP
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: status.hostIP
    - name: ISTIO_CPU_LIMIT
      valueFrom:
        resourceFieldRef:
          divisor: "0"
          resource: limits.cpu
    - name: SERVICE_ACCOUNT
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: spec.serviceAccountName
    - name: ISTIO_META_WORKLOAD_NAME
      value: istio-ingressgateway
    - name: ISTIO_META_OWNER
      value: kubernetes://apis/apps/v1/namespaces/istio-system/deployments/istio-ingressgateway
    - name: ISTIO_META_MESH_ID
      value: cluster.local
    - name: TRUST_DOMAIN
      value: cluster.local
    - name: ISTIO_META_UNPRIVILEGED_POD
      value: "true"
    - name: ISTIO_META_CLUSTER_ID
      value: Kubernetes
    - name: ISTIO_META_NODE_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: spec.nodeName
    image: docker.io/istio/proxyv2:1.19.3
    imagePullPolicy: IfNotPresent
    name: istio-proxy
    ports:
    - containerPort: 15021
      protocol: TCP
    - containerPort: 8080
      protocol: TCP
    - containerPort: 8443
      protocol: TCP
    - containerPort: 31400
      protocol: TCP
    - containerPort: 15443
      protocol: TCP
    - containerPort: 15090
      name: http-envoy-prom
      protocol: TCP
    readinessProbe:
      failureThreshold: 30
      httpGet:
        path: /healthz/ready
        port: 15021
        scheme: HTTP
      initialDelaySeconds: 1
      periodSeconds: 2
      successThreshold: 1
      timeoutSeconds: 1
    resources:
      limits:
        cpu: "2"
        memory: 1Gi
      requests:
        cpu: 10m
        memory: 40Mi
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
      privileged: false
      readOnlyRootFilesystem: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/workload-spiffe-uds
      name: workload-socket
    - mountPath: /var/run/secrets/credential-uds
      name: credential-socket
    - mountPath: /var/run/secrets/workload-spiffe-credentials
      name: workload-certs
    - mountPath: /etc/istio/proxy
      name: istio-envoy
    - mountPath: /etc/istio/config
      name: config-volume
    - mountPath: /var/run/secrets/istio
      name: istiod-ca-cert
    - mountPath: /var/run/secrets/tokens
      name: istio-token
      readOnly: true
    - mountPath: /var/lib/istio/data
      name: istio-data
    - mountPath: /etc/istio/pod
      name: podinfo
    - mountPath: /etc/istio/ingressgateway-certs
      name: ingressgateway-certs
      readOnly: true
    - mountPath: /etc/istio/ingressgateway-ca-certs
      name: ingressgateway-ca-certs
      readOnly: true
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-x5g2h
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: cluster-1
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    fsGroup: 1337
    runAsGroup: 1337
    runAsNonRoot: true
    runAsUser: 1337
  serviceAccount: istio-ingressgateway-service-account
  serviceAccountName: istio-ingressgateway-service-account
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - emptyDir: {}
    name: workload-socket
  - emptyDir: {}
    name: credential-socket
  - emptyDir: {}
    name: workload-certs
  - configMap:
      defaultMode: 420
      name: istio-ca-root-cert
    name: istiod-ca-cert
  - downwardAPI:
      defaultMode: 420
      items:
      - fieldRef:
          apiVersion: v1
          fieldPath: metadata.labels
        path: labels
      - fieldRef:
          apiVersion: v1
          fieldPath: metadata.annotations
        path: annotations
    name: podinfo
  - emptyDir: {}
    name: istio-envoy
  - emptyDir: {}
    name: istio-data
  - name: istio-token
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          audience: istio-ca
          expirationSeconds: 43200
          path: istio-token
  - configMap:
      defaultMode: 420
      name: istio
      optional: true
    name: config-volume
  - name: ingressgateway-certs
    secret:
      defaultMode: 420
      optional: true
      secretName: istio-ingressgateway-certs
  - name: ingressgateway-ca-certs
    secret:
      defaultMode: 420
      optional: true
      secretName: istio-ingressgateway-ca-certs
  - name: kube-api-access-x5g2h
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2023-10-29T16:21:31Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2023-10-29T16:21:41Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2023-10-29T16:21:41Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2023-10-29T16:21:31Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://caa7f68f1fb981f69dc451fd8d2e9945211617033527ee374b41ea8b607d4a88
    image: istio/proxyv2:1.19.3
    imageID: docker-pullable://istio/proxyv2@sha256:47b08c6d59b97e69c43387df40e6b969a2c1979af29665e85698c178fceda0a4
    lastState: {}
    name: istio-proxy
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2023-10-29T16:21:39Z"
  hostIP: 192.168.72.3
  phase: Running
  podIP: 10.244.0.7
  podIPs:
  - ip: 10.244.0.7
  qosClass: Burstable
  startTime: "2023-10-29T16:21:31Z"

```

## **NOTE!!!! Multiple kubernetes service listening on SAME port
```bash
istio-tutorials  kubectl get svc -o wide
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE   SELECTOR
details           ClusterIP   10.108.42.14    <none>        9080/TCP            17m   app=details
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP             36m   <none>
loki              ClusterIP   10.111.68.161   <none>        3100/TCP,9095/TCP   17m   app.kubernetes.io/component=single-binary,app.kubernetes.io/instance=loki,app.kubernetes.io/name=loki
loki-memberlist   ClusterIP   None            <none>        7946/TCP            17m   app.kubernetes.io/instance=loki,app.kubernetes.io/name=loki,app.kubernetes.io/part-of=memberlist
productpage       ClusterIP   10.111.207.76   <none>        9080/TCP            17m   app=productpage
ratings           ClusterIP   10.104.100.84   <none>        9080/TCP            17m   app=ratings
reviews           ClusterIP   10.97.242.9     <none>        9080/TCP            17m   app=reviews

## Above output shows that you can have multiple k8s services exposting SAME port (9080 in above exammple).
```
---

# 6. Istio Objects detailed descriptions

## 6.1 Gateway
 Gateway Captures the traffic comming via Ingress-Gateway-Controller, on specific host/port. But the traffic then remains at Gateway with no where to go (that's where Virtual-service comes into play)
https://istio.io/latest/docs/tasks/traffic-management/ingress/

```bash
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "bookinfo.app"   #This will allow only traffic comming to bookinfo.app host
EOF

// OR do it for all hosts

kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
EOF
```

 ## 6.1 Virtual Service (with Gateway)
Virtual Service (when used with Gateway) routes the traffic associated with specific gateways/hosts, and mathcing rules to a destination (a kubernetes service on specific port)
A virtual service lets you configure how requests are routed to a service within an Istio service mesh, building on the basic connectivity and discovery provided by Istio and your platform.
https://istio.io/latest/docs/concepts/traffic-management/#introducing-istio-traffic-management

```bash
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "bookinfo.app"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
EOF

// OR do it for all hosts

kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
EOF
```

## 6.1 Example: Use GW for a specific host
```bash
 istio-tutorials  cat istio-basics/workspace/traffic-management/ingress/bookinfo-gateway-with-specific-host.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  # The selector matches the ingress gateway pod labels.
  # If you installed Istio using Helm following the standard documentation, this would be "istio=ingress"
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 8080
      name: http
      protocol: HTTP
    hosts:
    - "bookinfo.app"   #This will allow only traffic comming to bookinfo.app host
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "bookinfo.app"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage


## first, delete existing gateway and vs
 istio-tutorials  kubectl delete gw bookinfo-gateway
gateway.networking.istio.io "bookinfo-gateway" deleted
 istio-tutorials  kubectl delete vs bookinfo
virtualservice.networking.istio.io "bookinfo" deleted
 
## Now recreate the GW & VS using a specific host
 istio-tutorials  kubectl apply -f istio-basics/workspace/traffic-management/ingress/bookinfo-gateway-with-specific-host.yaml 
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created


## Lets test it
 istio-tutorials  curl -s -HHost:bookinfo.app http://$INGRESS_HOST:$INGRESS_PORT/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
```

## 6.2 Virtual Service (between K8s deployment and k8s services)
A VirtualService defines a set of traffic routing rules to apply when a host is addressed. Each routing rule defines matching criteria for traffic of a specific protocol. If the traffic is matched, then it is sent to a named destination service (or subset/version of it) defined in the registry. The source of traffic can also be matched in a routing rule. This allows routing to be customized for specific client contexts.

You can think of virtual services as how you route your traffic to a given destination, and then you use destination rules to configure what happens to traffic for that destination.

When Virtual Services are used between K8s deployment and K8s services, it can OPTIOANLLY use subset (DestinationRule) to play with weight/loadbalancing/other aspects before the traffic is eventually landed on a pod. Virtual Services can be used without creating explicit destination rules.
https://istio.io/latest/docs/tasks/traffic-management/request-routing/

```bash
 istio-tutorials  cat istio-basics/workspace/traffic-management/request-routing/virtual-service1.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - bookinfo.com
  http:
  - match:
    - uri:
        prefix: /reviews
    route:
    - destination:
        host: reviews
  - match:
    - uri:
        prefix: /ratings
    route:
    - destination:
        host: ratings


 istio-tutorials  cat istio-basics/workspace/traffic-management/request-routing/virtual-service2.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v3         


 istio-tutorials  cat istio-basics/workspace/traffic-management/request-routing/virtual-service3.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 99
    - destination:
        host: reviews
        subset: v2
      weight: 1                                                                                                                                                                                                   
 
```

## 6.3 DestinationRule
Along with virtual services, destination rules are a key part of Istio’s traffic routing functionality. You can think of virtual services as how you route your traffic to a given destination, and then you use destination rules to configure what happens to traffic for that destination. Destination rules are applied after virtual service routing rules are evaluated, so they apply to the traffic’s “real” destination.

In particular, you use destination rules to specify named service subsets, such as grouping all a given service’s instances by version. You can then use these service subsets in the routing rules of virtual services to control the traffic to different instances of your services.

Destination rules also let you customize Envoy’s traffic policies when calling the entire destination service or a particular service subset, such as your preferred load balancing model, TLS security mode, or circuit breaker settings. You can see a complete list of destination rule options in the Destination Rule reference https://istio.io/latest/docs/reference/config/networking/destination-rule/ .

```bash
 istio-tutorials  cat istio-basics/workspace/traffic-management/request-routing/destination-rule1.yaml 
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews.default.svc.cluster.local
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2


 istio-tutorials  cat istio-basics/workspace/traffic-management/request-routing/destination-rule2.yaml
# Traffic policies defined at the service-level can be overridden at a subset-level
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews.default.svc.cluster.local
  trafficPolicy:
    loadBalancer:
      simple: LEAST_REQUEST
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN


 istio-tutorials  cat istio-basics/workspace/traffic-management/request-routing/destination-rule3.yaml
# the following rule configures a client to use mutual TLS for connections to upstream database cluster.
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: db-mtls
spec:
  host: mydbserver.prod.svc.cluster.local
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /etc/certs/myclientcert.pem
      privateKey: /etc/certs/client_private_key.pem
      caCertificates: /etc/certs/rootcacerts.pem


 istio-tutorials  cat istio-basics/workspace/traffic-management/request-routing/destination-rule4.yaml
# The following rule configures a client to use TLS when talking to a foreign service whose domain matches *.foo.com.
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: tls-foo
spec:
  host: "*.foo.com"
  trafficPolicy:
    tls:
      mode: SIMPLE


 istio-tutorials  cat istio-basics/workspace/traffic-management/request-routing/destination-rule5.yaml
# The following rule configures a client to use Istio mutual TLS when talking to rating services.
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: ratings-istio-mtls
spec:
  host: ratings.prod.svc.cluster.local
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
```

## 6.4 Demo Traffic Routing (VS,DestinationRule, Timout, fault-injection, retry, CircutBreaking) 
### Refer to https://istio.io/latest/docs/tasks/traffic-management/


## **Tool for making concurrent requests
#### We can use a tool (h2load) to make concurrent requests . It can be helpful while testing circut breaking with istio.
```bash
>h2load -n1000 -c1 'http://'"$INGRESS_HOST"':'"$INGRESS_PORT"'/productpage'
  // Above will filer n(1000) requests with c(1) concurrent clients

>h2load -n1000 -c3 'http://'"$INGRESS_HOST"':'"$INGRESS_PORT"'/productpage'
    // Above will filer n(1000) requests with c(3) concurrent clients    
```

## 6.5 Manual injection of istio-proxy
kubectl apply -f <(istioctl kube-inject -f istio/istio-1.19.3/samples/httpbin/httpbin.yaml ) -n bar


# 7. Security

## 7.1 Authentication
  - RequestAuthentication (user to service, mainly using JWT + MTLS)
    ** End user access to services. For this istio supports RequestAuthenitcation using JWT validation or any other OpenID Connect providers(ORY Hydra/KeyCloak/Firebase/Google/etc..).
  - PeerAuthentication (service to service, mainly using MTLS)
    ** With MTLS , each service gets its own identity which is enforced using key/cert pair. Certification generation and distribution are all managed automatically by istiod.

## 7.2 Authorization
  - AuthorizationPolicy provides a flexible approach to access control (allow/deny/custom/audit)for inbound traffic. we can control which service can reach which service.
  --> Should/Can add a namespace level deny-all
  --> And then allow only for specific case


## 7.3 Demo Authenitcation

### Create a new App in a new NS. This new app will call the product page.

```bash
// Create a new ns+app
 istio-tutorials  kubectl apply -f <(istioctl kube-inject -f istio/istio-1.19.3/samples/httpbin/httpbin.yaml ) -n bar
serviceaccount/httpbin created
service/httpbin created
deployment.apps/httpbin created

 istio-tutorials  kubectl get all -n bar
NAME                           READY   STATUS    RESTARTS   AGE
pod/httpbin-5c44d89fd9-m9r7l   2/2     Running   0          66s

NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/httpbin   ClusterIP   10.110.202.100   <none>        8000/TCP   66s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/httpbin   1/1     1            1           66s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/httpbin-5c44d89fd9   1         1         1       66s
```

### Make a call from this new app to product page (which is in default ns). It should work as no security/MTLS is enabled yet.

```bash
 istio-tutorials  kubectl exec "$(kubectl get pod -l app=httpbin -n bar -o jsonpath={.items..metadata.name})" -c istio-proxy -n bar -- curl "http://productpage.default:9080" -s -o /dev/null -w "%{http_code}\n"

200
```

### Now lets configure a PeerAuthenticationPolicy for default ns with mutual TSL mode STRICT.
```bash
kubectl apply -f - <<EOF
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "default"
spec:
  mtls:
    mode: STRICT
EOF

```

### Make a call again from this new app to product page (which is in default ns). It should fail (because we just added PeerAuthentication with mtls mode STRICT). 
#### Workloads that do not apply the same mode can not reach our application.

```bash
 istio-tutorials  kubectl exec "$(kubectl get pod -l app=httpbin -n bar -o jsonpath={.items..metadata.name})" -c istio-proxy -n bar -- curl "http://productpage.default:9080" -s -o /dev/null -w "%{http_code}\n"     ✘

000
command terminated with exit code 56
```

## 7.4 Demo Authorization

### First make sure that product page is up and running and gw+vs are present
### then apply the allow-nothing policy for default namespace

```bash
 istio-tutorials  cat istio-basics/workspace/security/authorization/authorizationpolicy-allow-nothing-ns-wide.yaml 
# See more at https://istio.io/latest/docs/tasks/security/authorization/authz-http/

# Below policy creates an allow-nothing policy in the default namespace. 
# The policy doesn’t have a selector field, which applies the policy to every workload in the default namespace. 
# The spec: field of the policy has the empty value {}. That value means that no traffic is permitted, effectively denying all requests.


apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: allow-nothing
  namespace: default
spec:
  {}

 istio-tutorials  kubectl apply -f  istio-basics/workspace/security/authorization/authorizationpolicy-allow-nothing-ns-wide.yaml
authorizationpolicy.security.istio.io/allow-nothing created
 
 // Note: You might need to wait for at least 15 seconds to have it applied.
 
  istio-tutorials  curl "http://$INGRESS_HOST:$INGRESS_PORT/productpage"
RBAC: access denied
 
```

### Now add the policy for Viewer page only (but nothing else, even nothing will be allowed between different services within default ns)

```bash
 istio-tutorials  cat istio-basics/workspace/security/authorization/authorizationpolicy-allow-with-scope.yaml     
# See more at https://istio.io/latest/docs/tasks/security/authorization/authz-http/

# Run the following command to create a productpage-viewer policy to allow access with GET method to the productpage workload. 
# The policy does not set the from field in the rules which means all sources are allowed, effectively allowing all users and workloads:


apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: "productpage-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  action: ALLOW
  rules:
  - to:
    - operation:
        methods: ["GET"]
 istio-tutorials  kubectl apply -f  istio-basics/workspace/security/authorization/authorizationpolicy-allow-with-scope.yaml
authorizationpolicy.security.istio.io/productpage-viewer created

 istio-tutorials  curl "http://$INGRESS_HOST:$INGRESS_PORT/productpage"
<!DOCTYPE html>

// Notice that only the product page GET is working, nothing else (including rest of the service) is available.
```


### Now allow details-page access from productPage only 
```bash
 istio-tutorials  cat istio-basics/workspace/security/authorization/authorizationpolicy-allow-with-scope2.yaml             
# See more at https://istio.io/latest/docs/tasks/security/authorization/authz-http/

# Run the following command to create the details-viewer policy to allow the productpage workload, which issues requests using the cluster.local/ns/default/sa/bookinfo-productpage 
# service account, to access the details workload through GET methods:


apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: "details-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: details
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]    


 istio-tutorials  kubectl apply -f  istio-basics/workspace/security/authorization/authorizationpolicy-allow-with-scope2.yaml
authorizationpolicy.security.istio.io/details-viewer created

// Notice that now GET request from productPage towards details page is allowed.
```


### Now allow reviews-page access from productPage only 
```bash
  istio-tutorials  cat istio-basics/workspace/security/authorization/authorizationpolicy-allow-with-scope3.yaml              
# See more at https://istio.io/latest/docs/tasks/security/authorization/authz-http/

# Run the following command to create the reviews-viewer policy to allow the productpage workload, which issues requests using the cluster.local/ns/default/sa/bookinfo-productpage 
# service account, to access the details workload through GET methods:


apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: "reviews-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]                                                                                                                                                     
 istio-tutorials  kubectl apply -f  istio-basics/workspace/security/authorization/authorizationpolicy-allow-with-scope3.yaml
authorizationpolicy.security.istio.io/reviews-viewer created


// Notice that now GET request from productPage towards reviews page is allowed.
```


### Now allow ratings-page access from reviews only 
```bash
 istio-tutorials  cat istio-basics/workspace/security/authorization/authorizationpolicy-allow-with-scope4.yaml              
# See more at https://istio.io/latest/docs/tasks/security/authorization/authz-http/

# Run the following command to create the ratings-viewer policy to allow the reviews workload, which issues requests using the cluster.local/ns/default/sa/bookinfo-productpage 
# service account, to access the details workload through GET methods:


apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: "ratings-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: ratings
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-reviews"]
    to:
    - operation:
        methods: ["GET"]                                                                                                                                                       
 istio-tutorials  kubectl apply -f  istio-basics/workspace/security/authorization/authorizationpolicy-allow-with-scope4.yaml
authorizationpolicy.security.istio.io/ratings-viewer created

// Now everything is working
```


# 8. Useful commands

```bash
// set path to be able to invoke istioctl
>export PATH=$PWD/istio/istio-1.19.3/bin:$PATH

// see all possible options for istioctl
>istioctl --help

// show log levels for individual properties
>istioctl proxy-config log productpage-v1-564d4686f-4t24x 

// set log levels  to debug for individual properties
>istioctl proxy-config log productpage-v1-564d4686f-4t24x --level=rbac:debug,http:debug,filter:debug

// set log levels back to default(warning) for individual properties
>istioctl proxy-config log productpage-v1-564d4686f-4t24x --level=rbac:warning,http:warning,filter:warning
```



# 9. Egress: Accessing external services ( ServiceEntry usage)
https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/

Because all outbound traffic from an Istio-enabled pod is redirected to its sidecar proxy by default, accessibility of URLs outside of the cluster depends on the configuration of the proxy. 
By default, Istio configures the Envoy proxy to pass through requests for unknown services. Although this provides a convenient way to get started with Istio, configuring stricter control is usually preferable.

**External services are those services that are not defined in Istio’s internal service registry.

This task shows you how to access external services in three different ways:

- Allow the Envoy proxy to pass requests through to services that are not configured inside the mesh.
- Configure service entries to provide controlled access to external services.
- Completely bypass the Envoy proxy for a specific range of IPs.


```bash
// Prerequisites

// 1. deploy an app with curl in it
 istio-tutorials  kubectl apply -f istio/istio-1.19.3/samples/sleep/sleep.yaml 
serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created

// 2. Set the SOURCE_POD environment variable to the name of your source pod:
 istio-tutorials  export SOURCE_POD=$(kubectl get pod -l app=sleep -o jsonpath='{.items..metadata.name}')

```
## 9.1 Envoy passthrough to external services

Istio has an installation option, meshConfig.outboundTrafficPolicy.mode, that configures the sidecar handling of external services, that is, those services that are not defined in Istio’s internal service registry. If this option is set to ALLOW_ANY, the Istio proxy lets calls to unknown services pass through. If the option is set to REGISTRY_ONLY, then the Istio proxy blocks any host without an HTTP service or service entry defined within the mesh. ALLOW_ANY is the default value, allowing you to start evaluating Istio quickly, without controlling access to external services. You can then decide to configure access to external services later.

```bash
// Run the following command to verify that meshConfig.outboundTrafficPolicy.mode option is set to ALLOW_ANY or is omitted:
// You should see ALLOW_ANY or no output  (default ALLOW_ANY)
 istio-tutorials  kubectl get istiooperator installed-state -n istio-system -o jsonpath='{.spec.meshConfig.outboundTrafficPolicy.mode}'
 istio-tutorials  

// Make a couple of requests to external HTTPS services from the SOURCE_POD to confirm successful 200 responses:
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sI https://www.google.com | grep  "HTTP/"; 
HTTP/2 200 
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sI https://edition.cnn.com | grep  "HTTP/";    
HTTP/2 200 

// You can see from above that you successfully sent egree traffic from your mesh. 
```

But this simple approach to access external services, has the drawback that you lose Istio monitoring and control for traffic to external services. The next section shows you how to monitor and control your mesh’s access to external services.


## 9.2 Controlled access to external services (Using ServiceEntry)

Using Istio ServiceEntry configurations, you can access any publicly accessible service from within your Istio cluster. This section shows you how to configure access to an external HTTP service, httpbin.org, as well as an external HTTPS service, www.google.com without losing Istio’s traffic monitoring and control features.

### Change to the blocking-by-default policy
To demonstrate the controlled way of enabling access to external services, you need to change the meshConfig.outboundTrafficPolicy.mode option from the ALLOW_ANY mode to the REGISTRY_ONLY mode.

**In our case, since we used istioctl to install istio,  we can either unistall+reinstall with additional arguments or just run the below one to UPDATE the existing installation

```bash
istio-tutorials  istioctl install --set profile=demo -y --set meshConfig.outboundTrafficPolicy.mode=REGISTRY_ONLY
✔ Istio core installed 
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete 
  Made this installation the default for injection and validation.

// Now check if the flag is set correctly, you should see REGISTEY_ONLY
 istio-tutorials  kubectl get istiooperator installed-state -n istio-system -o jsonpath='{.spec.meshConfig.outboundTrafficPolicy.mode}'
REGISTRY_ONLY%   
```

Make a couple of requests to external HTTPS services from the SOURCE_POD to verify they are now blocked:


```bash
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sI https://www.google.com | grep  "HTTP/" 
command terminated with exit code 35
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sI https://edition.cnn.google.com | grep  "HTTP/" 
command terminated with exit code 35
```



### 9.2.1 Access an external HTTP service (Using ServiceEntry)

```bash

// First create a ServiceEntry to allow access to an external HTTP service ( httpbin.org in this example ).

 istio-tutorials  kubectl apply -f - << EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: httpbin-ext
spec:
  hosts:
  - httpbin.org
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
  location: MESH_EXTERNAL
EOF
serviceentry.networking.istio.io/httpbin-ext created

// Make a request to the external HTTP service from SOURCE_POD:

 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sS http://httpbin.org/headers
{
  "headers": {
    "Accept": "*/*", 
    "Host": "httpbin.org", 
    "User-Agent": "curl/8.4.0", 
    "X-Amzn-Trace-Id": "Root=1-654227ab-67f5212834285ec65e42c1d7", 
    "X-B3-Sampled": "1", 
    "X-B3-Spanid": "ec849552efd0f7a0", 
    "X-B3-Traceid": "bcda1e63318f6759ec849552efd0f7a0", 
    "X-Envoy-Attempt-Count": "1", 
    "X-Envoy-Decorator-Operation": "httpbin.org:80/*", 
    "X-Envoy-Peer-Metadata": "ChkKDkFQUF9DT05UQUlORVJTEgcaBXNsZWVwChoKCkNMVVNURVJfSUQSDBoKS3ViZXJuZXRlcwodCgxJTlNUQU5DRV9JUFMSDRoLMTAuMjQ0LjAuNjcKGQoNSVNUSU9fVkVSU0lPThIIGgYxLjE5LjMKoQEKBkxBQkVMUxKWASqTAQoOCgNhcHASBxoFc2xlZXAKJAoZc2VjdXJpdHkuaXN0aW8uaW8vdGxzTW9kZRIHGgVpc3RpbwoqCh9zZXJ2aWNlLmlzdGlvLmlvL2Nhbm9uaWNhbC1uYW1lEgcaBXNsZWVwCi8KI3NlcnZpY2UuaXN0aW8uaW8vY2Fub25pY2FsLXJldmlzaW9uEggaBmxhdGVzdAoaCgdNRVNIX0lEEg8aDWNsdXN0ZXIubG9jYWwKHwoETkFNRRIXGhVzbGVlcC05NDU0Y2M0NzYtaDVoZDIKFgoJTkFNRVNQQUNFEgkaB2RlZmF1bHQKSQoFT1dORVISQBo+a3ViZXJuZXRlczovL2FwaXMvYXBwcy92MS9uYW1lc3BhY2VzL2RlZmF1bHQvZGVwbG95bWVudHMvc2xlZXAKGAoNV09SS0xPQURfTkFNRRIHGgVzbGVlcA==", 
    "X-Envoy-Peer-Metadata-Id": "sidecar~10.244.0.67~sleep-9454cc476-h5hd2.default~default.svc.cluster.local"
  }
}

// Note the headers added by the Istio sidecar proxy: X-Envoy-Decorator-Operation.


// Check the log of the sidecar proxy of SOURCE_POD:

 istio-tutorials  kubectl logs "$SOURCE_POD" -c istio-proxy | tail
...
[2023-11-01T10:25:47.107Z] "GET /headers HTTP/1.1" 200 - via_upstream - "-" 0 1152 560 559 "-" "curl/8.4.0" "3c555887-4af0-9379-9796-19666fee3267" "httpbin.org" "54.204.25.77:80" outbound|80||httpbin.org 10.244.0.67:55696 54.204.25.77:80 10.244.0.67:55694 - default

Note the entry related to your HTTP request to httpbin.org/headers.
```

### 9.2.2 Access an external HTTPS service (Using ServiceEntry)

```bash
// Create a ServiceEntry to allow access to an external HTTPS service.

 istio-tutorials  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: google
spec:
  hosts:
  - www.google.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
  location: MESH_EXTERNAL
EOF
serviceentry.networking.istio.io/google created


// Make a request to the external HTTPS service from SOURCE_POD:
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sSI https://www.google.com | grep  "HTTP/"
HTTP/2 200 


// Check the log of the sidecar proxy of SOURCE_POD:
 istio-tutorials  kubectl logs "$SOURCE_POD" -c istio-proxy | tail
2023-11-01T10:02:06.861650Z     info    xdsproxy        connected to upstream XDS server: istiod.istio-system.svc:15012
[2023-11-01T10:06:41.930Z] "- - -" 0 UH - - "-" 0 0 0 - "-" "-" "-" "-" "-" BlackHoleCluster - 142.251.36.4:443 10.244.0.67:49062 - -
[2023-11-01T10:06:55.205Z] "- - -" 0 UH - - "-" 0 0 0 - "-" "-" "-" "-" "-" BlackHoleCluster - 151.101.131.5:443 10.244.0.67:39056 - -
[2023-11-01T10:10:20.538Z] "- - -" 0 - - - "-" 789 6096 75 - "-" "-" "-" "-" "151.101.131.5:443" PassthroughCluster 10.244.0.67:43514 151.101.131.5:443 10.244.0.67:43512 - -
[2023-11-01T10:10:24.982Z] "- - -" 0 - - - "-" 844 5873 131 - "-" "-" "-" "-" "142.251.36.4:443" PassthroughCluster 10.244.0.67:53918 142.251.36.4:443 10.244.0.67:53916 - -
[2023-11-01T10:11:02.701Z] "- - -" 0 UH - - "-" 0 0 0 - "-" "-" "-" "-" "-" BlackHoleCluster - 142.251.36.4:443 10.244.0.67:54754 - -
[2023-11-01T10:11:06.157Z] "- - -" 0 UH - - "-" 0 0 0 - "-" "-" "-" "-" "-" BlackHoleCluster - 151.101.195.5:443 10.244.0.67:57876 - -
[2023-11-01T10:25:47.107Z] "GET /headers HTTP/1.1" 200 - via_upstream - "-" 0 1152 560 559 "-" "curl/8.4.0" "3c555887-4af0-9379-9796-19666fee3267" "httpbin.org" "54.204.25.77:80" outbound|80||httpbin.org 10.244.0.67:55696 54.204.25.77:80 10.244.0.67:55694 - default
2023-11-01T10:34:04.963394Z     info    xdsproxy        connected to upstream XDS server: istiod.istio-system.svc:15012
[2023-11-01T10:36:14.663Z] "- - -" 0 - - - "-" 844 5875 101 - "-" "-" "-" "-" "142.251.36.4:443" outbound|443||www.google.com 10.244.0.67:59172 142.251.36.4:443 10.244.0.67:59170 www.google.com -

// Note the entry related to your HTTPS request to www.google.com.

```

### 9.2.3 Manage traffic to external services

Similar to inter-cluster requests, Istio routing rules can also be set for external services that are accessed using ServiceEntry configurations. In this example, you set a timeout rule on calls to the httpbin.org service.

```bash
// From inside the pod being used as the test source, make a curl request to the /delay endpoint of the httpbin.org external service:

 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- time curl -o /dev/null -sS -w "%{http_code}\n" http://httpbin.org/delay/5
200
real    0m 5.65s
user    0m 0.00s
sys     0m 0.00s

// The request should return 200 (OK) in approximately 5 seconds.


// Use kubectl to set a 3s timeout on calls to the httpbin.org external service:
 
 istio-tutorials  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin-ext
spec:
  hosts:
    - httpbin.org
  http:
  - timeout: 3s
    route:
      - destination:
          host: httpbin.org
        weight: 100
EOF
virtualservice.networking.istio.io/httpbin-ext created


// Wait a few seconds, then make the curl request again:

 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- time curl -o /dev/null -sS -w "%{http_code}\n" http://httpbin.org/delay/5
504
real    0m 3.01s
user    0m 0.00s
sys     0m 0.00s

// This time a 504 (Gateway Timeout) appears after 3 seconds. Although httpbin.org was waiting 5 seconds, Istio cut off the request at 3 seconds.
```


### 9.2.4 Cleanup the controlled access to external services


```bash
 istio-tutorials  kubectl get vs,se
NAME                                             GATEWAYS               HOSTS             AGE
virtualservice.networking.istio.io/bookinfo      ["bookinfo-gateway"]   ["*"]             2d13h
virtualservice.networking.istio.io/httpbin-ext                          ["httpbin.org"]   4m19s

NAME                                           HOSTS                LOCATION        RESOLUTION   AGE
serviceentry.networking.istio.io/google        ["www.google.com"]   MESH_EXTERNAL   DNS          17m
serviceentry.networking.istio.io/httpbin-ext   ["httpbin.org"]      MESH_EXTERNAL   DNS          28m

 istio-tutorials  kubectl delete vs httpbin-ext
virtualservice.networking.istio.io "httpbin-ext" deleted

 istio-tutorials  kubectl delete se google httpbin-ext
serviceentry.networking.istio.io "google" deleted
serviceentry.networking.istio.io "httpbin-ext" deleted

 istio-tutorials  kubectl get vs,se                   
NAME                                          GATEWAYS               HOSTS   AGE
virtualservice.networking.istio.io/bookinfo   ["bookinfo-gateway"]   ["*"]   2d13h
```

## 9.3 Direct access to external services

If you want to completely bypass Istio for a specific IP range, you can configure the Envoy sidecars to prevent them from intercepting external requests. This option should be considered as a last resort when, for performance or other reasons, external access cannot be configured using the sidecar. Read more at https://istio.io/latest/docs/tasks/traffic-management/egress/egress-control/#direct-access-to-external-services



# 10. Egress: Accessing external services using Egress Gateway
As per Istio documentation, Egress Gateway configuration does not work with minikube. But actually it does
Please refer to documentation at https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/

The Accessing External Services task shows how to configure Istio to allow access to external HTTP and HTTPS services from applications inside the mesh. There, the external services are called directly from the client sidecar. 

This example also shows how to configure Istio to call external services, although this time indirectly via a dedicated egress gateway service.

Istio uses ingress and egress gateways to configure load balancers executing at the edge of a service mesh. An ingress gateway allows you to define entry points into the mesh that all incoming traffic flows through. Egress gateway is a symmetrical concept; it defines exit points from the mesh. Egress gateways allow you to apply Istio features, for example, monitoring and route rules, to traffic exiting the mesh.

Consider an organization that has a strict security requirement that all traffic leaving the service mesh must flow through a set of dedicated nodes. These nodes will run on dedicated machines, separated from the rest of the nodes running applications in the cluster. These special nodes will serve for policy enforcement on the egress traffic and will be monitored more thoroughly than other nodes.

Another use case is a cluster where the application nodes don’t have public IPs, so the in-mesh services that run on them cannot access the Internet. Defining an egress gateway, directing all the egress traffic through it, and allocating public IPs to the egress gateway nodes allows the application nodes to access external services in a controlled way.


## 10.1 Egress gateway for HTTP traffic

Lets first confirm that traffic from within mesh to an external service is not open

```bash
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sI https://edition.cnn.com/politics 
command terminated with exit code 35
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sSL -o /dev/null -D - http://edition.cnn.com/politics     
HTTP/1.1 502 Bad Gateway
date: Wed, 01 Nov 2023 11:45:45 GMT
server: envoy
content-length: 0

// Notice the -L flag of curl which instructs curl to follow redirects.

```

### 10.1.1 Now, create a ServiceEntry for edition.cnn.com

```bash
 istio-tutorials  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cnn
spec:
  hosts:
  - edition.cnn.com
  ports:
  - number: 80
    name: http-port
    protocol: HTTP
  - number: 443
    name: https
    protocol: HTTPS
  resolution: DNS
EOF
serviceentry.networking.istio.io/cnn created
```
### 10.1.2 Verify that your ServiceEntry was applied correctly by sending an HTTP request to http://edition.cnn.com/politics.

```bash
 istio-tutorials kubectl exec "$SOURCE_POD" -c sleep -- curl -sSL -o /dev/null -D - http://edition.cnn.com/politics
HTTP/1.1 301 Moved Permanently
...
location: https://edition.cnn.com/politics
...

HTTP/2 200 
content-type: text/html; charset=utf-8
```
### 10.1.3 Create an egress Gateway for edition.cnn.com, port 80, and a destination rule for traffic directed to the egress gateway.

```bash
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - edition.cnn.com
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: egressgateway-for-cnn
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: cnn
EOF

gateway.networking.istio.io/istio-egressgateway created
destinationrule.networking.istio.io/egressgateway-for-cnn created
```

### 10.1.4 Define a VirtualService to direct traffic from the sidecars to the egress gateway and from the egress gateway to the external service:

```bash
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: direct-cnn-through-egress-gateway
spec:
  hosts:
  - edition.cnn.com
  gateways:
  - istio-egressgateway
  - mesh
  http:
  - match:
    - gateways:
      - mesh
      port: 80
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: cnn
        port:
          number: 80
      weight: 100
  - match:
    - gateways:
      - istio-egressgateway
      port: 80
    route:
    - destination:
        host: edition.cnn.com
        port:
          number: 80
      weight: 100
EOF

virtualservice.networking.istio.io/direct-cnn-through-egress-gateway created
```

### 10.1.5 Resend the HTTP request to http://edition.cnn.com/politics
The output should be the same as in the step 2.
```bash
 istio-tutorials kubectl exec "$SOURCE_POD" -c sleep -- curl -sSL -o /dev/null -D - http://edition.cnn.com/politics
HTTP/1.1 301 Moved Permanently
...
location: https://edition.cnn.com/politics
...

HTTP/2 200 
content-type: text/html; charset=utf-8
...
```
### 10.1.6 Check the log of the istio-egressgateway pod for a line corresponding to our request. If Istio is deployed in the istio-system namespace, the command to print the log is:



```bash
istio-tutorials  kubectl logs -l istio=egressgateway -c istio-proxy -n istio-system | tail
[2023-11-01T11:59:56.883Z] "GET /politics HTTP/2" 301 - via_upstream - "-" 0 0 41 41 "10.244.0.67" "curl/8.4.0" "085155d7-7ec9-9a5f-8049-a821b27f72c4" "edition.cnn.com" "151.101.3.5:80" outbound|80||edition.cnn.com 10.244.0.61:35544 10.244.0.61:8080 10.244.0.67:36494 - -
2023-11-01T12:00:03.719701Z     info    xdsproxy        connected to upstream XDS server: istiod.istio-system.svc:15012
```
Note that you only redirected the traffic from port 80 to the egress gateway. The HTTPS traffic to port 443 went directly (via SE) to edition.cnn.com.

### 10.1.7 Cleanup HTTP Gateway

```bash
 istio-tutorials  kubectl delete gateway istio-egressgateway
gateway.networking.istio.io "istio-egressgateway" deleted
 istio-tutorials  kubectl delete serviceentry cnn
serviceentry.networking.istio.io "cnn" deleted
 istio-tutorials  kubectl delete virtualservice direct-cnn-through-egress-gateway
virtualservice.networking.istio.io "direct-cnn-through-egress-gateway" deleted
 istio-tutorials  kubectl delete destinationrule egressgateway-for-cnn
destinationrule.networking.istio.io "egressgateway-for-cnn" deleted
```

## 10.2 Egress gateway for HTTPS traffic

In this section you direct HTTPS traffic (TLS originated by the application) through an egress gateway. You need to specify port 443 with protocol TLS in a corresponding ServiceEntry, an egress Gateway and a VirtualService.

Lets first confirm that traffic from within mesh to an external service is not open

```bash
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sI https://edition.cnn.com/politics 
command terminated with exit code 35
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sSL -o /dev/null -D - http://edition.cnn.com/politics     
HTTP/1.1 502 Bad Gateway
date: Wed, 01 Nov 2023 11:45:45 GMT
server: envoy
content-length: 0

// Notice the -L flag of curl which instructs curl to follow redirects.
```

### 10.2.1 Define a ServiceEntry for edition.cnn.com:
```bash
 istio-tutorials  kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cnn
spec:
  hosts:
  - edition.cnn.com
  ports:
  - number: 443
    name: tls
    protocol: TLS
  resolution: DNS
EOF

serviceentry.networking.istio.io/cnn created
```

### 10.2.2 Verify that your ServiceEntry was applied correctly by sending an HTTPS request to https://edition.cnn.com/politics.

```bash
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sSL -o /dev/null -D - https://edition.cnn.com/politics
HTTP/2 200 
content-type: text/html; charset=utf-8
...
```
### 10.2.3 Create an egress Gateway for edition.cnn.com, a destination rule and a virtual service to direct the traffic through the egress gateway and from the egress gateway to the external service.

```bash
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: 443
      name: tls
      protocol: TLS
    hosts:
    - edition.cnn.com
    tls:
      mode: PASSTHROUGH
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: egressgateway-for-cnn
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: cnn
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: direct-cnn-through-egress-gateway
spec:
  hosts:
  - edition.cnn.com
  gateways:
  - mesh
  - istio-egressgateway
  tls:
  - match:
    - gateways:
      - mesh
      port: 443
      sniHosts:
      - edition.cnn.com
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: cnn
        port:
          number: 443
  - match:
    - gateways:
      - istio-egressgateway
      port: 443
      sniHosts:
      - edition.cnn.com
    route:
    - destination:
        host: edition.cnn.com
        port:
          number: 443
      weight: 100
EOF

gateway.networking.istio.io/istio-egressgateway created
destinationrule.networking.istio.io/egressgateway-for-cnn created
virtualservice.networking.istio.io/direct-cnn-through-egress-gateway created
```

### 10.2.4 Send an HTTPS request to https://edition.cnn.com/politics. The output should be the same as before.
```bash
 istio-tutorials  kubectl exec "$SOURCE_POD" -c sleep -- curl -sSL -o /dev/null -D - https://edition.cnn.com/politics
HTTP/2 200 
content-type: text/html; charset=utf-8
...
```
### 10.2.5 Check the log of the egress gateway’s proxy. 

```bash
 istio-tutorials  kubectl logs -l istio=egressgateway -n istio-system
 ...
[2023-11-01T12:14:50.188Z] "- - -" 0 - - - "-" 791 2480891 369 - "-" "-" "-" "-" "151.101.195.5:443" outbound|443||edition.cnn.com 10.244.0.61:58100 10.244.0.61:8443 10.244.0.67:60140 edition.cnn.com -
```

### 10.2.6 Cleanup HTTPS gateway

```bash
 istio-tutorials  kubectl delete serviceentry cnn
serviceentry.networking.istio.io "cnn" deleted
 istio-tutorials  kubectl delete gateway istio-egressgateway
gateway.networking.istio.io "istio-egressgateway" deleted
 istio-tutorials  kubectl delete virtualservice direct-cnn-through-egress-gateway
virtualservice.networking.istio.io "direct-cnn-through-egress-gateway" deleted
 istio-tutorials  kubectl delete destinationrule egressgateway-for-cnn
destinationrule.networking.istio.io "egressgateway-for-cnn" deleted
```

