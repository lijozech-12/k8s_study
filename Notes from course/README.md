# Notes from udemy CKA preparation course.

## Core Concepts

### Cluster Architecture

Host application as containers 
worker nodes are the ships that load container.
control ship = master node
All stored in ETCD Cluster.
information about the containers in ETCD. in key:value format.

crain which is used to load the ships are kube-scheduler. it is used for where it be placed.

Controllers
1. Node-controllers
2. Replication-controller
3. Controller-Manager


Kube-apiserver = the primary management controller k8s. It is responsible for orchestrating different things in kube-apiserver.

Docker is the container runtime engine. also support rkt and contneird

Kubelet = captain of each ship(node) manage container running in the 

kube-proxy =  it wil ensure all the rules are in place and running.

### Docker vs Container-d

k8s is build to orchestate docker at first.
k8s introduced is CRI(continaer runtime interface) to work with it
If it adhere to OCI standards. Open Container Initiative. imagespec and runtimespec
dockershim is as temporary fix to work with docker since it's doest have OCI standards.

containerD also part of docker. if we don't docker. we car use crt
nerdctl docker-liek cli for containerD.

```bash
# docker command and it equivalent nerdctl command
docker 

nerdctl

docker run --name redis redis:alpine

nerdctl run --name redis redis:alpine

docker run --name webserver -p 80:80 -d nginx

nerdctl run --name webserver -p 80:80 -d nginx
```

crictl provide a CLI for CRI compatible container. it's only used for debugging purposes

```bash
crictl

crictl pull busybox

crictl images

crictl ps -a

crictl exec -i -t <container-id> ls

crictl logs <container-id> # to see logs 

crictl pods
```

### ETCD

key-value store

etcd is a distrbuted reliable key-value store that is simple, secure & fast.

when you install and run etcd it will start in port 2379. `./etcd`
we can store key-value as 
```bash
# it will create a key value pair
./etcdctl set key1 value1
./etcdctl put key1 value1 #v3

# to get value
./etcdctl get key1 #v2 and v3


# to check which api version is set to work with v2 or v3. default version in 3

./etcdctl --version

# to list all the commands
./etcdctl

# to set version
ETCDCTL_API=3 ./etcdctl version

# or set for entire session
export ETCDCTL_API=3 ./etcdctl version


```

### ETCD in Kubernetes
 
stores all the changes. etcd is also deployed according to our setup

### ETCD commands


ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3.  By default its set to use Version 2. Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:

```bash
etcdctl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set

```


Whereas the commands are different in version 3

```bash
etcdctl snapshot save 
etcdctl endpoint health
etcdctl get
etcdctl put
```


To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3



When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.



Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

```bash
--cacert /etc/kubernetes/pki/etcd/ca.crt     
--cert /etc/kubernetes/pki/etcd/server.crt     
--key /etc/kubernetes/pki/etcd/server.key
```

So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:

```bash
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"
```

### kube-apiserver

primary management controller.

when we run a command like `create pod`
1. Authenticate user
2. Validate Request
3. Retrieve data from etcd
4. Update ETCD
5. Scheduler it finds te best node to pod
6. kubelet in that specified node create the pod and update it in etcd

everything goes to kube-apiserver

### kube controller manager.

1. watch status 
2. Remediate situation


Node-controller check the status of nodes and if a node unreaacble it will destroyed and created more

replication controller. check status and create again
### kube Scheduler

decide which pod goes to which node.
right container ends up right node.
it depends on resource requirements. It first filter nodes, the rand nodes

### Kubelet

likes the captain of the ship
it will place the container in a pod. 

### kube proxy

Pod network is internal network in k8s. 
it runs is each pod. it will look for services. when a service is created it will create rules to connect to it.

### Pods

Pod is single instance of application. 
it will increase a pod in each node.

Multi container pods is only for helper containers. can't have same type of pod.

how to deploy pod ``` kubectl run nginx ```

### Pods with yaml

All k8s defintion contains four things
1. apiVersion: version we are going to create
2. kind: type or resources we are going to create
3. metadata: data about object like name, labels
4. spec: It's a dictionary

``` bash
kubectl create -f pod-definition.yaml

kubectl get pods # to see the pods

kubectl describe pod <pod name> #check the event session to know what's the error

# to check no of pods
kubectl get pods

# to check the help
kubectl run --help

# to create a pod easiest way
kubectl run nginx --image=nginx

# to help with deployment
kubectl run redis --image=redis123 --dry-run=client -o yaml
#it will output the defintion in yaml format. we can copy paste it.

kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
# To store it in a yaml file

kubectl edit pod <podname> # to edit the pod

# or edit the yaml file
kubectl apply -f redis.yaml to apply the changes

```

pod.yaml file
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx
  - name: busybox  #if you want more than one container
    image: busybox  
```

### replica-set

replication controller(old) replicset(new) helps to run multiple instance of container in our node.
it will 

rc-definition.yml
```bash
apiVersion: v1
kind: ReplicationController
metadata: #meta data of replicaset
  name: myapp-rc
  labels:
    app: myapp
    type: frontend
spec:
  template: #to have a template of pod we are creating. the metadata and spec are copied from pod
    metadata: #metadata of pod
      name: nginx
      labels:
        app: nginx
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
  replicas: 3 #how much replicas are needed   
```

```bash
# find replication controller.
kubectl create -f rc-defintion.yml
# to see different replicas
kubctl get replicationcontroller

kubectl get pods
```

##### replicaset

replicaset-defintion.yml
```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata: #meta data of replicaset
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template: #to have a template of pod we are creating. the metadata and spec are copied from pod
    metadata: #metadata of pod
      name: myapp-pod
      labels:
        app: myapp
        tier: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3 #how much replicas are needed 
  selector: #Major difference between replicaset and replication controller.
    matchLabels:
      type: front-end  

```

Replicaset monitor and deploy if their is any failure. 
this lables will help to monitor everthing and create new one if their is anyfailure.

to scale the replicaset. update the number in replicas `replicas: 6`
then run ``` kubectl replace -f replicaset-defintion.yml```

```bash
kubectl create -f replicaset-defintion.yml
kubectl get replicaset
kubectl delete replicaset <name of replicaset>
kubectl replace -f replicaset-definition.yml #to replace the replicaset
kubectl scale --replicas=6 -f replicaset-defintion.yml #using definition file

kubectl scale --replicas=6 replicaset myapp-replicaset # using type and name

# the above command changes the size of replicasets. but the number will stay the same in file
```

### Deployments

It is have bigger scope than replica-set. it will update if there is any updates in the replicaset.

```bash
apiVersion: apps/v1
kind: Deployment
metadata: #meta data of replicaset
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template: #to have a template of pod we are creating. the metadata and spec are copied from pod
    metadata: #metadata of pod
      name: myapp-pod
      labels:
        app: myapp
        tier: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3 #how much replicas are needed 
  selector: #Major difference between replicaset and replication controller.
    matchLabels:
      type: front-end  

```

deployment create replicaset and thus pods

```bash
kubectl get all #see all the resources created

kubectl get deployments

#Create an NGINX Pod

kubectl run nginx --image=nginx

#Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

kubectl run nginx --image=nginx --dry-run=client -o yaml

# Create a deployment

kubectl create deployment --image=nginx nginx

# Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

#Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

#Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

kubectl create -f nginx-deployment.yaml



#OR

#In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

### Services

Connectivity betwen different users, database, frontend. losecoupling between different service.

listen to port on pod and forward that traffic into a port in the node.


service types.
1. nodeport port for a node
2. ClusterIp: ip address for node
3. LoadBalancer: for an ip

port on pod: target port. 
port: the port in service
port on the node where the node is accessed externally : nodeport

service-definition.yaml
```bash
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports: #it's an arry and -targetPort indicate the first port in an array.
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: front-end # labels from the pod defintion file
```

It will create a service spanning accross the nodes and make the service available

### ClusterIP

Ip of the clusterIP
service-definition.yaml
```bash
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: clusterIP
  ports: #it's an arry and -targetPort indicate the first port in an array.
  - targetPort: 80
    port: 80
  selector:
    app: myapp
    type: back-end # labels from the pod defintion file
```

### Loadbalancer

If we are hosted our application in multiple nodes. the data is accessible to all the nodeIps.
Users need a single URL. 

```bash
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  ports: #it's an arry and -targetPort indicate the first port in an array.
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: front-end
```

### NameSpaces

it is used for isolation
kube-system, Default, kube-public the namespace created in k8s
We can define policies, quota of resources each of namespaces.

to connect to another service in another namespace.
```bash
db-service.dev.svc.cluster.local

kubectl get pods --namespace=kube-system #list resources of another namespace.

kubectl create -f pod-definition.yml --namespace=dev #to create pod in another namespace

```

Name space in pod-defintion.yml file
```bash
apiVersion: v1
kind: Pod

metadata:
  name: myapp-pod
  namespace: dev
  labels:
    app: myapp
    type: front-end
  spec:
    containers:
    - name: nginx-container
      image: nginx

```

creation of namespace using namespace-dev.yml file

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```bash
kubectl crete -f namespace-dev.yml

kubectl create namespace dev # for easy creation

kubectl config set-context $(kubectl config current-context) --namespace=dev # to set current context to dev

kubectl get pods

kubect get pods --namespace=default

kubectl get pods --namespace=prod

kubectl get pods --all-namespaces #to see pods in all namespaces
```

For defining **Resource Quota** for a namespace
compute-quota.yaml

```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

### Imperative and Declarative

Imperative: saying all the  steps to be done: running commands
Declarative: only giving the .yaml file to work

```bash
# Create Objects
kubectl run --image=nginx nginx
kubectl create deployment --image=nginx nginx
kubectl expose deployment nginx --port 80

#Update Objects
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18

kubectl replace -f nginx.yaml  #after editing the .yaml file
kubectl replace --force -f nginx.yaml  #forcefully replacing it
```

Declartive

```bash

#create objects

kubectl apply -f nginx.yaml #it will check if the resource is there or not. if ther it skip otherwise create it

kubectl apply -f /path/to/config-files #create all the resources from config files

#Update Objects

kubectl apply -f nginx.yaml
```

user imperative command in exam and declarative in working

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
https://kubernetes.io/docs/reference/kubectl/conventions/

### Kubectl apply command

takes the local file. 
if configuration doesn't ther create live object configuration file
and saved json format(last applied configuration). so 3 things

live object configuration

## Scheduling

easiest wayt to schedule a node without schedular is set `nodeName: node02` filled.

pod-definition.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
    nodeName: node02
```

pod-bind-definiton.yaml
```bash
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```


### Labels and Selectors

Labels give names to items like
class: bird, kind: Domestic, Color: Green

selector helps to filter these items
class = mammal
color = Green

```bash
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
```

Once the pod is created to select the pod created use
```bash
# to select certain type of app
kubectl get pods --selector app=App1
```

replicaset-definition.yaml
```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels: #labels of replicaset
    app: App1
    function: Front-end
  annotations: # used to store version number. phonenumber emails for integrations purposes.
    buildversion: 1.34
spec:
  replicas: 3
  selector: # used to select the pods
    matchLabels:
      app: App1
  template:
    metadata:
      labels: # labels of pod. selectors use this
        app: App1
        function: Front-end
    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp

```

service-definition.yaml

```bash
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector: # select App1 in pods 
    app: App1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
```

### Taints and Toleration

A node with tainted those pods have tolerations can place that in particular place.
**Taint effects**
1. NoSchedule
2. PreferNoSchedule
3. NoExecute

```bash
kubectl taint nodes <node-name> key=value:tain-effect

kubectl taint nodes node1 app=blue:NoSchedule #nothing will be scheduled
```

pod-defintion.yml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx

  tolerations: # same as app=blue:NoSchedule we used in the taint command in ""
  - key:"app"
    operator:"Equal"
    value:"blue"
    effect:"NoSchedule"
```

tells the node which pod to be accepted

### Node Selectors

first we need to label nodes.
```
kubectl label nodes node-1 size=Large
```

pod-defintion.yml
```bash
apiVersion:
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor

  nodeSelector:
    size: Large
```

When pod is created it placed in the node.

### node affinity

To ensure the particular pods placed in desired nodes.

pod-definition.yml
```bash
apiVersion:
kind:
metadata:
  name: myapp-pod
spec:

containers:
- name: data-processor
  image: data-processor
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: Exists
```
node affinity rules
```bash
requiredDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution

#planned
requiredDuringSchedulingrequiredDuringExecution
```

duringscheduling: it will check if node is there not during scheduling
duringexection: important if there a change of labels of nodes

### Taints/Tolerations and Node Affinity.

Taints and Tolerations helps to which pod should not be placed. But the pod required can be placed in other nodes.

NodeAffinity helps to which pod should be placed. but other pods can be placed in a particular node.

We need to use both of them to get a desired results.

### Resource Requirments and Limits

pod-definition.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    resources:
      requests: #the minimum requirment
        memory: "1Gi"
        cpu: 1
      limits: # the maximum requirment
        memory: "2Gi"
        cpu: 2
```

Requesting a certain amount of memory and cpu. 
Cpu can't go beyond limit. But memory can go beyond the limit. but it go beyond the memory. but if the pod cis consuming the memory more than it's limits it is terminated with OOM(Out Of Memory).

best practice is having requests but not limits. since a resource can use the resource if no one is using. but the same time can return the extra resources when other pods which have request set is needed.

limit-range-cpu.yaml 
```bash
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default: #limit
      cpu: 500m
    defaultRequest: #request
      cpu: 500m
    max: #limit
      cpu: "1"
    min: #request
      cpu: 100m
    type: Container
```
default for all the containers and pods created in namespace without having a set of values set.

limit-range-memory.yaml
```bash
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500mi
    type: Container
```

Resource quota at namespace level. for pods without limits and requests.

```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```

### Daemon Sets

Automatically added or destroyed in the pod. Daemon set is added.

If you want to add a monitoring solution, Logs Viewer or any pods like that daemon set is the perfect companion for that. No need worry about adding or removing agents from the pod because daemon set will take care of that.

replicaset-definition.yaml
```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        name: monitoring-agent
```

daemon-set-definiton.yaml this is similar to replicaset-definition.yaml file but diff in kind
```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        name: monitoring-agent
```

```
kubectl get daemonsets
kubectl describe daemonsets monitoring-daemon
```

### Static Pods

kubelet wait for kube-apiserver to give instruction to create cluster. What if there is no kube-apiserver or master.
kubelet can manage nodes independly.
kubelet can create their on pods(pods only no replicasets/daemonsets). We can configure a kubelet to read from a directory called /etc/kubernetes/manifests. Where there is stored all the pod definition files as pod1.yaml/pod2.yaml.

![pod are saved](images/image.png)

kubelet works on pod level can only understands pods. 

since static pods are depended on control plane. we can add control place etcd and everything

`kubectl get pods -n kube-system`

static pods deploy **control place** components as static pods
daemonset deply monitoring agents, Logging agents on node

both are ignored by the Kube-schedular

### Multiple Schedular

we can create other schedular so some of them can be used as main schedular or schedular for specific pods.

my-scheduler-2-config.yaml
```bash
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler-2
leaderElection:
  leaderElect: true
  resourceNamespace: kube-system
  resourceName: lock-object-my-scheduler
```

we can create scheduler by downloading the binary and starting different schedulers as service.

```
# ExecStart have the location of kube-scheduler
ExecStart=/usr/local/bin/kube-scheduler \\ --config=/etc/kubernetes/config/my-scheduler-2.yaml

#config as location of custom kube-scheduler name
```

deploying scheduler as pod
my-custom-scheduler.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --config=/etc/kubernetes/my-scheduler-config.yaml

    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
  ```

** Using custom scheduler

pod-definition.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  
  schedulerName: my-custom-scheduler #scheduler name
```

### Scheduler Profiles

 






