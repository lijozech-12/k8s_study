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

### Pods in Kubernetes

A Pod is a collection of application containers and volumes running in the same execution environment.
It is the smallest deployable artifact in a k8s cluster.

### Thinking with Pods

What should I put in a POD ?

* WordPress container + MySQL database container = WordPress instance. This kind of pod is actually an antipattern for pod construction.
* the scaling startegy for both them are different and won't fit for eachother

Ask will these contaners work correctly if they land on different machines ? if the answer is no . The pod is the correct grouping fo the containers. 
If answer is yes, Using multiple pods is probably the correct solution.

### Creating a POD

* simplest way to create a pod with ``` kubectl run ```

``` shell
kubectl run kuard --generator=run-pod/v1 --image=gcr.io/kuar-demo/kuard-amd64:blue 
```

* see the node running

``` shell
kubectl get pods
```
* delete the node

``` shell
kubectl delete pods/kuard
```

### Creating Pod Manifest

* Can be created using YAML or JSON. But YAML is generally preferred since it's slightly more human-editable and support comments.
* It should be treated as the same way as source code.

``` yaml
# for creating kurad-pod.yaml
apiVersion: v1
kind: Pod
metadata:
name: kuard
spec:
containers:
- image: gcr.io/kuar-demo/kuard-amd64:blue
name: kuard
ports:
- containerPort: 8080
name: http
protocol: TCP
```

* Written recode of desired state is the best practice in the long run, especially for large teams with many applications.

### Running  Pods

* use `kubectl  apply` apply command to launch a single instance of kurd

``` shell
kubectl apply -f kurad-pod.yaml
```

##### How it works

1. The pod manifest will be submitted to the kubernetes API Server. 
2. The k8s system will then schedule that pod to run on a healthy node in the cluster, `kubelet` daemon will monitor it.

### Listing Pods

    kubectl get pods

### Pod Details

    kubectl describe pods kuard

* To get more details on pod
It contains 
basic information
![alt text](Images/describe_1.png)
Containers running in the pod
![alt text](Images/describe_2.png)
events related to the pod.
![alt text](Images/describe_3.png)

### Deleting a POD

* Delete either by name
    kubectl delete pods/kuard
* Or can use the same file to delete
    kubectl delete -f kuard-pod.yaml

* When you delete a pod it won't immediately killed. It has a termination grace period of 30 seconds. When pod is in that state it no longer receives new requests. This grace period is important for reliability becuase it allows the pods to finish any active requests that it may be in the middle of processing before it is terminated.

* when you delete a pod the data stored also gets deleted. If you want to persist data across mulitple instances of a pod, we need persistent Volumes

### Accessing Your POD

For various reasons including not limited too..

* Want to load the web service running in the pod
* To see logs to debug a problem
* To execute other commands inside the Pod to help debug

##### Getting more Information with Logs

* To understand what the application doing
``` shell
    kubectl logs kuard # download the current logs
    # -f flag to stream
    # --previous flag to get logs from previous instance of the container
```

* While it's useful for occasional debugging of containers in prodcution env, it's generally useful to use log aggregation service.

##### Running Commands in your Container with exec

``` shell
kubectl exec kuard -- date # to run a command

# -t flag for interactive session

kubectl exec -it kuard -- ash

```

#### Copying Files to and from containers

* Copying files from one container to another is antipattern. We should treat the contenct of a container as immutable.

* But sometimes it's the most immediate way to stop the bleeding and restore the service to health. since it is quicker than building, pushing, and rolling out a new image.

* But when you stop bleeding it is critically important that you immediately go and do the image build and rollout. Otherwise the local changes is made to the container is forgotten and overwritten by next rollout.

### Health Checks

* When we run a application as container in k8s it keep it alive by using a process `health check`. It checks the process is running or not. else it's restarts everything.

* But it's insuffieciet for deadlocks and all. since health check things that application is still running.

* To address this k8s introduced health checks for application `liveness`.

* It applies application specific logic, like loading a web page. Since it's application specific, we have to define them in pod manifest

##### Liveness Probe


