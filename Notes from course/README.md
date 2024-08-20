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

When pod si created it will be in Scheduling Queue. and pod with higher prioriy. Then filtering face what can be placed or not. Then Scoring phase every node is scored according to it, which node have more space left. Then Pod is Binded with the Node.

Scheduling Queue: PrioritySort
Filtering: NodeResourcesFit, NodeName, NodeUnschedulable
Scoring: NodeResourcesFit, ImageLocality
Binding: DefaultBinder

All the plugins are added to extenstions

https://github.com/kubernetes/community/blob/master/contributors/devel/sig-scheduling/scheduling_code_hierarchy_overview.md

https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/

https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/

https://stackoverflow.com/questions/28857993/how-does-kubernetes-scheduler-work


### Monitor k8s cluster

Install metric server from github or using minikube.

### Managing Application Logs.

If we are creating two containers using a single yaml file.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
  - name: image-processor
    image: some-image-processor
```

To see logs of kubectl use.
```
kubectl logs -f event-simulator-pod event-simulator
```

## Application Lifecycle Management

### Rolling Updates and Rollbacks

Every time container got changes a new rollout happeing

```
# to see the status of a deployment
kubectl rollout status deployment/myapp-deployment

# To see the history of the rollouts
kubectl rollout history deployment/myapp-deployment

kubectl apply -f deployment-definition.yml
# to make changes in deployment

kubectl rollout undo deployment/myapp-deployment
# to rollout(going back) to the last change
```

**Deployment Strategy**

Recreate: destroy all the pods at once and create everything it will have app down
Rolling update: destroy and update pods one-by-one. zero downtime.

```bash
#create a new pod
kubectl create -f deployment-definition.yml

#getting new update
kubectl get deployments

#updating the pods/deployment
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1 #it won't update the file only the deployment

kubectl apply -f deployment-definintion.yml #after updating the .yml file

#Status
kubectl rollout status deployment/myapp-deployment

kubectl rollout history deployment/myapp-deployment

#rollback to before state
kubectl rollout undo deployment/myapp-dep1


```

Their will be already a replicaset is present. and another replicaset is created. Every pod is created one by one then each pod is deleted one by one. Until all the pods are duplicated.

### ENV Variables in K8s

Direct way of specifying variables

pod-definition.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    env:
    - name: APP_COLOR
      value: pink
```

Using ConfigMap
```
env:
- name: APP_COLOR
  valueFrom:
    configMapKeyRef:
```

Using Secrets
```
env:
- name: APP_COLOR
  valueFrom:
    secretKeyRef:
```

### ConfigMaps

Where varaibles are stored as keyvalue pairs. When application is created these values are injected as keyvalue pairs into the pod-definition file.

Create configmaps --->>> Inject them into 

```bash

#impertive way
kubectl create configmap \
  app-config --from-literal=APP_COLOR=blue \
             --from-literal=APP_MOD=prod

# using configmaps stored in a file
kubectl create configmap \
  app-config --from-file=app_config.properties
```

```
kubectl create -f config-map.yaml

kubectl get configmaps
```
config-map.yaml
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

Injecting config maps to pods

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
    envFrom: #injecting env variables
    - configMapRef:
        name: app-config
```

### Secrets in Application

used to save secrets in a encoded format inside the pod. 

```bash
#imperative method

kubectl create secret generic <secret-name> --from-literal=<key>=<value>

kubectl create secret generic \
  app-secret --from-literal=DB_Host=mysql \
             --from-literal=DB_User=root


kubectl create secret generic \
    <secret-name> --from-file=<path-to-file>

kubectl create secret generic \
    app-secret --from-file=app_secret.properties

```

declarative approach
secret-data.yaml
```bash
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: <encoded format>
  DB_User: <encoded format>
  DB_Password: <encoded format>
```

```
kubectl create -f secret-data.yaml

kubectl get secrets

kubectl describe secrets
```

How to convert data into encoded format.

```
# IN linux
# to encode the value
echo -n '<text needed to be encoded> | base 64


# to decode the value
echo -n '<encoded text>' | base64 --decode

```

**Inject into pod**

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
    envFrom:
    - secretRef:
        name: app-secret
```

### Encrypting Secret Data at Rest 

Don't push secrets into github or anything. Since it's only encoded not encrypted.

### Multi Container Pods

LOG Agent and WEB Server

some containers needs to work together in pod. So we will give containers that can start together and destroyed together.

pod-definition.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
  - name: log-agent
    image: log-agent
```

### Init Containers

In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. 
For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times.

But their is times when you need to pulls a code or binary from a repository that will be used by the main web application. that is a task that will be run only on time when the pod is first created. This containers are called **initContainers**.

An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section,  like this:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```
If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/


## Cluster Maintenance

### OS upgrade

If you want to remove pods and add new pods.

```
kubectl drain node-1  #it will mark the node unschedulable, then schedule all the pod in that node to another node. We can update os or other things in the nodes


kubectl cordon node-2 #mark node unschedulable. It won't delete existing pod. but new pods are not schedulable


kubectl uncordon node-1  #mark node schedulabe
```

### Cluster Upgrade process

K8s support recent 3 updates fro it's components. Like kubeapiserver, kubelet.
they will suppot only recent 3 updates. Like 1.10, 1.11. 1.12 extra.
To upgrade nodes first pass the pods to another nodes then upgrade the node or create an upgraded node then pass the pods.

Updating nodes simultaneously make issues like downtime extra. But updating one by one doesn't cause that issue.

We can only go one minor version at a time.

guess our current version is 1.11
```bash
apt-get upgrade -y kubeadm=1.12.0-00 #get kubeadm

#upgrade kubeadm

kubeadm upgrade apply v1.12.0

# upgrade kubelet

apt-get upgrade -y kubelet=1.12.0-00

systemctl restart kubelet

kubectl get nodes
```

Steps to upgrade a node
```bash

# first drain the node and make unschedulabel. node name is node-1
kubectl drain node-1

# upgrade the kubeadm 
apt-get upgrade -y kubeadm=1.12.0-00 #version to which you want to upgrade

#upgrade kubelet
apt-get upgrade -y kubelet=1.12.0-00

#kubeadm upgrade
kubeadm upgrade node config --kubelet-version v1.12.0

# restart kubelet
systemctl restart kubelet

# To make it schedulabel
kubectl uncordon node-1

# do this to all the nodes node-1, node-2, node-3
```



``` kubeadm upgrade plan ``` we need upgrade kubeadm to upgrade the cluster

cluster upgrade.
``` cat /etc/*release*```

kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

### Backup and Restore Methods


```bash
# query kubeapiserver and save the resource in 
# backup using kube apiserver
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml

# we can also backup using etcd
#ETCD cluster backupt. It is hosted in etcd cluster
# It will backup the snapshot
ETCDCTL_API=3 etcdctl \
    snapshot save snapshot.db

service kube-apiserver stop 

# saves the snapshot and save it in data dir 
ETCDCTL_API=3 etcdctl \
    snapshot save snapshot.db \
    --data-dir /var/lib/etcd-from-backup

# after that make changes to etcd configuration file etcd.service

# then reload the service daemon and restart the etcd cluster
systemctl daemon-reload
service etcd restart
service kube-apiserver start


ETCDCTL_API=3 etcdctl \
    snapshot save snapshot.db \

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

https://github.com/etcd-io/website/blob/main/content/en/docs/v3.5/op-guide/recovery.md

https://www.youtube.com/watch?v=qRPNuT080Hk

```


## Security

### Security Primitives in k8s

**Secure Hosts**

- password based authentication disabled
- SSH key based authentication

### Authentication


k8's doesn't store details internally it relay upon outside users and outside certificaition teams.

```bash
# we cannot create users or see users list like this
kubectl create user user1

kubectl list users

## but we can create and manage service accounts.
# Since the passwords and all managed by third party like LDAP

kubectl create serviceaccount sa1 #to create service accounts
```

all the request from admin or developers goes throught kube-apiserver and it's Authenticate User before processing the request.

kube-apiserver have static password file, static token file, certificates, identity services.


**Auth Mechnisms-Basic**

Create csv file with password, user and user-id
user-details.csv
```bash
password1,user1,user-id1
password2,user2,user-id2
password3,user3,user-id3
```
add this to kube-apiserver.service file
```bash
--basic-auth-file=user-details.csv
```
we can also have user-token-details.csv instead of that so that we can have tokenized password file.

### TLS certificates.

Securing k8s with TLS certificates and other cluster components.

### TLS Basics

certificate is used to have trust between two parties in a network.

symmetric encryption where we use the same key to encrypt and decrypt but the hacker can snipe into it and read the password.

Asymmetric Encryption. where uses two keys public and private.

If you want to have communication with a bank server. the bank send a public key with validation certificates from a authenticated certificate authority(CA). after that user uses this key to encrypt there symmetric key and send it to the bank. so that they can decrypt the password of the and other details of the users.

This symmetric is used for communication going forward.

Root Certificaties : used by CA
Server Certificates: Used by server
Client Certificates: in the Client

### TLS in k8s

The interactions between master and all the nodes need to be secured.

server components

kube-api server has apiserver.crt and apiserver.key / apiserver-kubelet-client.crt apiserver-kubelet-client.key
etcd server: etcdserver.crt etcdserver.key / apiserver-etcd-client.crt apiserver-etcd-client.key
kubelet server : kubelet.crt kubelet.key / kubelet-client.crt kubelet-client.key

client certificates for clients. 
admin admin.crt, admin.key
kubescheduler scheduler.crt scheduler.key
kube-controller-manger controller-manager.crt controler-manager.key
kube-proxy kube-proxy.crt kube-proxy.key

these 3 has key and crts

### viewing tls certificates

```bash
cat /etc/systemd/system/kube-apiserver.service
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

for viewing the certificate location after getting the certificate go to 
`/etc/kubernetes/pki/apiserver.crt`

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

### Certificate API

the key-cert pair that stored in server in cs server. 

1. Create Certificate Sigining Request Object.
``` openssl genrsa -out jane.key 2048 ```
2. Review Requests
``` openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr ```

administor takes the key and makes a certificate sigining requests objects
this object is created by using a manifest file using usual fields.
jane.csr -> `cat jane.csr | base64` -> encoded format
jane-csr.yaml
```bash
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  expirationSeconds: 600 #seconds
  usages:
  - digital signature
  - key encipherment
  - server auth
  request:
    <encoded text. then submit the request>
```
once the object is created all the request can be seen by admins by running.
`kubectl get csr`
3. Approve Requests
``` kubectl certificate approve <name of certificate(here jane)> ```
k8s signs the certificate using ca pairs and create a certificates for users.

4. Share Certs to Users.


``` 
# view the certificate using 
kubectl get csr jane -o yaml

# to decode the certificate 
echo "encoded text" | base64 --decode

# controller manager as
CSR-APPROVING CSR-SIGNING

cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```

### KubeConfig

client uses certificates to query list pods using curl

```bash
kubectl get pods
  --server my-kube-playground:6443
  --client-key admin.key
  --client-certificate admin.crt
  --certificate-authority ca.crt
```

typing this is tedious task so we can mobe this to a file.
KubeConfig File
config
```bash
  --server my-kube-playground:6443
  --client-key admin.key
  --client-certificate admin.crt
  --certificate-authority ca.crt
```
use the below command after that
```bash
kubectl get pods
  --kubeconfig config(name of the file)
```

usually it will look for file in `&HOME/.kube/config` where you don't need to specify the path.

kubeconfig file is in specific formats
clusters: to which you can access
Users: who as access to this cluster
Context: which user account can acces which cluster


#in below file the details are hidden
```bash
apiVersion: v1
kind: Config
current-context: dev-user@google
clusters:
- name: my-kube-playground
- name: productions
- name: google
contexts:
- name: my-kube-admin@my-kube-playground
- name: dev-user@google
- name: prod-user@proudction
users:
- name: my-kube-admin
- name: admin
- name: dev-user
- name: prod-user
```

# to see kubectl config command
```bash
#gives the main file in the location
kubectl config view 

# To see a custom file
kubectl config view --kubeconfig=my-custom-config

# To change the current context
kubectl config use-context prod-user@production

# config -h
kubectl config -h
```

### API Groups


