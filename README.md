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

* Everything in k8s is represented by a RESTful resource
* Each k8s object exists at a unique HTTP path example, https://your-k8s.com/api/v1/name‐spaces/default/pods/my-pod leads to the representation of a Pod in the default name‐space named my-pod

* To extracting specific fiels from kubectl

``` kubectl get pods my-pod -o jsonpath --template={.status.podIP} ```

* To view multiple objects of different types

``` kubectl get pods,services ```

* To get more details info

``` kubectl describe <resource-name> <obj-name> ```

* To see list of supported fields for each supported type of k8s object, use explain command

``` kubectl explain pods ```

### Creating, Updating, and Destroying Kubernetes Objects

* Objects in k8s API are represented as JSON or YAML files.
* This files can be returned by the server in response of query or posted to the server as part of an API request.
* We can use YAML or JSON files to create, update or delete objects on the k8s cluster.

``` kubectl apply -f obj.yaml ```

* No need to specify resource type of the object. it's obtained from the object file itself.
* To update the object

``` kubectl apply -f obj.yaml ```

* You can repeatedly use apply to reconcile state
* If you want what the apply command do without actually making the changes. Use ``` --dry-run ``` flag to print the objects to the terminal without actually sending them to the server.

* To make interactive edits instead of editing local file. use,

``` kubectl edit <resource-name> <obj-name> ```

* After you save the file, it will be automatically uploaded back to the k8s cluster.

The apply command also records the history of previous configurations in an annotation within the object. You can manipulate these records using ``` edit -last-applied ``` , ``` set-last-applied ``` and ``` view-last-applied ```

``` kubectl apply -f myobj.yaml view-last-applied ``` Show the last state that was applied
``` kubectl delete -f obj.yaml ``` to delete and obj
``` kubectl delete < resource-name > <obj-name> ``` to delete resource with type and name.

### Labeling and Annotating Objects

* tags for objects
* to update labels and annotations on any k8s use ``` label ``` and ``` anotate ``` command.

``` kubectl label pods bar color=red ```

* To remove label use ``` <label-name>- ```

``` kubectl label pods bar color- ```

### Debugging Commands

* To see logs of running conatainers ``` kubectl logs <pod-name> ```
* -c for multiple containers
* -f for following the logs

* ``` exec ``` to execute command in a running container

``` shell
 kubectl exec -it <pod-name> -- bash 
 ```

### Cluster management

* kubectl can be used to manage cluster itself. 

* ``` cordon ``` it will prevent pods being scheduled in a particular node in the future
* ``` drain ``` Remove existing pods from the node
* ``` uncordon ``` to re-enable pods scheduling onto the node.

It can be used for removing physical machine for repairs and upgrades. in that scenario we can use ``` kubectl cordon ``` followed by ``` kubectl drain ``` to safely remove machine from the cluster. we can use ``` kubectl uncordon ``` to renable it.
It's not required for something quick affecting a node (machine reboot). Only required when service out for long enough.

### Command Autocompletion

kubectl supports integration with your shell to enable tab completion for both
commands and resources. Depending on your environment, you may need to install
the bash-completion package before you activate command autocompletion. You
can do this using the appropriate package manager:
* macOS
``` shell 
brew install bash-completion
```
* CentOS/Red Hat
``` shell
 yum install bash-completion 
 ```
* Debian/Ubuntu
``` shell 
apt-get install bash-completion 
```
When installing on macOS, make sure to follow the instructions from brew about
how to activate tab completion using your ${HOME}/.bash_profile.
Once bash-completion is installed, you can temporarily activate it for your terminal
using:
``` shell 
source <(kubectl completion bash) 
```
To make this automatic for every terminal, add it to your ${HOME}/.bashrc file:
``` shell 
echo "source <(kubectl completion bash)" >> ${HOME}/.bashrc 
```
### Alternative ways of viewing a cluster

* You can also utilize other plugins like vs code, intelliJ, Eclipse

use ``` kubectl help ``` for more info.

![Kubectl help image](Images/kubectl_help.png)

## Pods

