# K8s notes

## Checking cluster status

``` kubectl version ```

version of local kubectl tool as well as version of k8s api server.

### Simple diagnostic for the cluster.

``` kubectl get componentstatuses ```

### Listing K8S Nodes

``` kubectl get nodes ```

To list all the nodes in the cluster

``` kubectl describe nodes kube1 ```

To get more informations about a specific node such as kube 1

## Cluster Components

### Kubernetes Proxy

The Kubernetes proxy is responsible for routing network traffic to load-balanced
services in the Kubernetes cluster. To do its job, the proxy must be present on every
node in the cluster.
If your
cluster runs the Kubernetes proxy with a DaemonSet, you can see the proxies by
running:

``` kubectl get daemonSets --namespace=kube-system kube-proxy ```

### Kubernetes DNS

* Provides naming discovery for the service that are defined in the cluster.
* Runs as replicated server
* No of DNS is depends upon size of cluster 1 or more..
* it also run as k8s deployment

``` kubectl get deployments --namespace=kube-system core-dns ```

* k8s service also performs load balancing for the DNS server

``` kubectl get services --namespace=kube-system core-dns ```

### Kubernetes UI

* to visulize the cluster

# Common kubectl commands

## Namespaces

* Used to organize objects in the cluster.
* Default kubectl command-line interact with default namespace

``` --namespace=mystuff```
``` -n``` flag
```--all-namespaces```

## Contexts

* if you want to change default namespace more permanently you can set it in context it will be in $HOME/.kube/config 

``` kubectl config set-context my-context --namespace=mystuff ```

creates new context, to start using it use

``` kubectl config use-context my-context ```

### Viewing Kubernetes API Objects


