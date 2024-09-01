# previous year questionaires and answers.

### 1.Take a backup of the etcd cluster and save it to /opt/etcd-backup.db

* search for etcd in k8s documentation. and search for backup their will be a command like this.
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>
  ```
  * for finding the ca-file, cert-file and  key-file search in the etcd manifest file

  ```bash
  k get pods -A #to get the pods and check etcd pod
  ```

* manifest file for all the resources are present in `/etc/kubernetes/manifests`
* to get the relevant details run the below command
```
cat etcd.yaml | grep file
```

* answer for the question is
```bash
controlplane $ ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key   snapshot save /opt/etcd-backup.db   
{"level":"info","ts":1724508868.6099272,"caller":"snapshot/v3_snapshot.go:68","msg":"created temporary db file","path":"/opt/etcd-backup.db.part"}
{"level":"info","ts":1724508868.6238513,"logger":"client","caller":"v3/maintenance.go:211","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1724508868.6240578,"caller":"snapshot/v3_snapshot.go:76","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":1724508868.7103605,"logger":"client","caller":"v3/maintenance.go:219","msg":"completed snapshot read; closing"}
{"level":"info","ts":1724508868.738288,"caller":"snapshot/v3_snapshot.go:91","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"4.4 MB","took":"now"}
{"level":"info","ts":1724508868.7383404,"caller":"snapshot/v3_snapshot.go:100","msg":"saved","path":"/opt/etcd-backup.db"}
Snapshot saved at /opt/etcd-backup.db
```

* To check the data run `cat /opt/etcd-backup.db`

### 2.Create a new deployment called nginx-deployment, with image nginx:1.16 and 2 replica. Next Upgrade the deployemtn to version 1.17 using rolling update. (record it)

* This question has 2 parts. 1st part is to create a deployment for that we need to use imperative commands.

```bash
controlplane $ k create deployment nginx-deployment --image nginx:1.16 --replicas 2
deployment.apps/nginx-deployment created

#to confirm the deployment
controlplane $ k get deployments.apps 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           82s
```

* Second is rolling update
** we can do this in two ways. one is using kubectl edit deployment command and editing the file and making changes
** searching for `rolling update` in k8s documentation
** we need to the name of the container
```bash
controlplane $ k get deployments.apps -o yaml
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "1"
    creationTimestamp: "2024-08-24T14:21:44Z"
    generation: 1
    labels:
      app: nginx-deployment
    name: nginx-deployment
    namespace: default
    resourceVersion: "5113"
    uid: abc4360b-47d9-4426-8866-930fb669a4e7
  spec:
    progressDeadlineSeconds: 600
    replicas: 2
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        app: nginx-deployment
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: nginx-deployment
      spec:
        containers:
        - image: nginx:1.16
          imagePullPolicy: IfNotPresent
          name: nginx
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
  status:
    availableReplicas: 2
    conditions:
    - lastTransitionTime: "2024-08-24T14:21:52Z"
      lastUpdateTime: "2024-08-24T14:21:52Z"
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: "True"
      type: Available
    - lastTransitionTime: "2024-08-24T14:21:45Z"
      lastUpdateTime: "2024-08-24T14:21:52Z"
      message: ReplicaSet "nginx-deployment-55d5db8cc8" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: "True"
      type: Progressing
    observedGeneration: 1
    readyReplicas: 2
    replicas: 2
    updatedReplicas: 2
kind: List
metadata:
  resourceVersion: ""
  ```

  You can see the nae of container and use it for setting the image

```bash
controlplane $ k set image deployment/nginx-deployment nginx=nginx:1.17 --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/nginx-deployment image updated



#to see whether recording it worked.

controlplane $ k rollout history deployment nginx-deployment 
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/nginx-deployment nginx=nginx:1.17 --record=true

```

### 3.Create a static pod named static-pod on the node01 node that uses the busybox image and the command sleep 2000.

https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

* If you want to run a pod in node01 you need to have the pod manifest in node01

```bash
controlplane $ k run static-pod --image busybox --dry-run=client -o yaml --command -- sleep 2000
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-pod
  name: static-pod
spec:
  containers:
  - command:
    - sleep
    - "2000"
    image: busybox
    name: static-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

* to run as static pod we need put this inside manifest folder inside the node we want to put this in. `/etc/kubernetes/manifests` 
```bash
#first sshing into the node

ssh node1

# go to the manifest folder
cd /etc/kubernetes/manifests

#copy the dry run file and create a file
nano static.yaml

exit #exit from node01 and go to controlplane
```

To check the running and working

```bash
controlplane $ k get pods -A
NAMESPACE            NAME                                       READY   STATUS    RESTARTS      AGE
default              static-pod-node01                          1/1     Running   0             54s
kube-system          calico-kube-controllers-75bdb5b75d-kh2wj   1/1     Running   2 (21m ago)   22d
kube-system          canal-rc7zk                                2/2     Running   2 (21m ago)   22d
kube-system          canal-rnmhk                                2/2     Running   2 (21m ago)   22d
kube-system          coredns-5c69dbb7bd-nzg9k                   1/1     Running   1 (21m ago)   22d
kube-system          coredns-5c69dbb7bd-pk85q                   1/1     Running   1 (21m ago)   22d
kube-system          etcd-controlplane                          1/1     Running   2 (21m ago)   22d
kube-system          kube-apiserver-controlplane                1/1     Running   2 (21m ago)   22d
kube-system          kube-controller-manager-controlplane       1/1     Running   2 (21m ago)   22d
kube-system          kube-proxy-6xsx9                           1/1     Running   2 (21m ago)   22d
kube-system          kube-proxy-vrwqx                           1/1     Running   1 (21m ago)   22d
kube-system          kube-scheduler-controlplane                1/1     Running   2 (21m ago)   22d
local-path-storage   local-path-provisioner-75655fcf79-s7bbc    1/1     Running   2 (21m ago)   22d
controlplane $ k get pods -o wide 
NAME                READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
static-pod-node01   1/1     Running   0          81s   192.168.1.4   node01   <none>           <none>
```

### 4.Create a new pod called super-pod with image busybox:1.28. Allow the pod to be able to set SYS_TIME. (The container should sleep for 4800seconds )(weightage: 8)

* seprate the question into 2 parts first part is to creating the pod and second part is to set SYS_TIME.

```bash
#creatig the pod
controlplane $ k run super-pod --image busybox:1.28 --command -- sleep 4800
pod/super-pod created

# to get pods in yaml format
controlplane $ k run super-pod --image busybox:1.28 --dry-run=client -o yaml --command -- sleep 4800
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: super-pod
  name: super-pod
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox:1.28
    name: super-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


# to store file in yaml format
k run super-pod --image busybox:1.28 --dry-run=client -o yaml --command -- sleep 4800 > q4.yaml

```

https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container

```bash
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-4
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/hello-app:2.0
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```
copy the securitycontext from above files.

```bash
controlplane $ k create -f q4.yaml 
pod/super-pod created

controlplane $ cat q4.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: super-pod
  name: super-pod
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox:1.28
    name: super-pod
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
controlplane $ 

```

To check whether everything worked fine

```bash
controlplane $ k get pods super-pod -o yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    cni.projectcalico.org/containerID: 36baea6d4e13dcf487814d70387bf43f713ce4cf256d137a6b32863570dcbc2b
    cni.projectcalico.org/podIP: 192.168.1.4/32
    cni.projectcalico.org/podIPs: 192.168.1.4/32
  creationTimestamp: "2024-08-24T15:23:34Z"
  labels:
    run: super-pod
  name: super-pod
  namespace: default
  resourceVersion: "3732"
  uid: 9664863d-378d-414e-a2d3-16f6d05ef795
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox:1.28
    imagePullPolicy: IfNotPresent
    name: super-pod
    resources: {}
    securityContext:
      capabilities:
        add:
        - SYS_TIME #added capabilities
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-l8pgd
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: node01
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
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
  - name: kube-api-access-l8pgd
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
    lastTransitionTime: "2024-08-24T15:23:37Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2024-08-24T15:23:34Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2024-08-24T15:23:37Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2024-08-24T15:23:37Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2024-08-24T15:23:34Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://3ebf736e8a603993ccb7698d8d5c754b41becc76a7747a8e151ca52d9d483486
    image: docker.io/library/busybox:1.28
    imageID: docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    lastState: {}
    name: super-pod
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2024-08-24T15:23:37Z"
  hostIP: 172.30.2.2
  hostIPs:
  - ip: 172.30.2.2
  phase: Running
  podIP: 192.168.1.4
  podIPs:
  - ip: 192.168.1.4
  qosClass: BestEffort
  startTime: "2024-08-24T15:23:34Z"

```

### 5.Create a nginx pod called nginx-resolved using image nginx, expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. use the image: busybox: 1.28 for dns lookup. Record results in nginx.svc and nginx.pod (weightabe 12)

```bash

#to create pod
controlplane $ k run nginx-resolver --image nginx
pod/nginx-resolver created

#to see pods created
controlplane $ k get pods
NAME             READY   STATUS              RESTARTS   AGE
nginx-resolver   0/1     ContainerCreating   0          4s
super-pod        1/1     Running             0          19m
controlplane $ k get pods -w #watch state
NAME             READY   STATUS    RESTARTS   AGE
nginx-resolver   1/1     Running   0          15s
super-pod        1/1     Running   0          20m

# creating services in the name of nginx-resolver-service
controlplane $ k expose nginx-resolver --name nginx-resolver-service 
error: the server doesn't have a resource type "nginx-resolver"
controlplane $ k expose nginx-resolver --name nginx-resolver-service --port 80
error: the server doesn't have a resource type "nginx-resolver"

#this is the correct command need to add port and pod resource type
controlplane $ k expose pod nginx-resolver --name nginx-resolver-service --port 80
service/nginx-resolver-service exposed


#to see the service
controlplane $ k get svc
NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes               ClusterIP   10.96.0.1    <none>        443/TCP   22d
nginx-resolver-service   ClusterIP   10.98.47.5   <none>        80/TCP    2m32s
```

* next part to check the nodes if it's working or not. dns lookup for that we need busybox

```bash
# to create dns and adding sleep 5000 so that it won't get into crashloop back error
controlplane $ k run testdns --image busybox:1.28 --command -- sleep 5000
pod/testdns created

# watching the pods
controlplane $ k get pods -w
NAME             READY   STATUS    RESTARTS   AGE
nginx-resolver   1/1     Running   0          12m
super-pod        1/1     Running   0          32m
testdns          1/1     Running   0          8s


#executing the command inside the testdns pod
controlplane $ k exec -it testdns -- nslookup nginx-resolver-service
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx-resolver-service
Address 1: 10.98.47.5 nginx-resolver-service.default.svc.cluster.local


# using the address got from running previous command
controlplane $ k exec -it testdns -- nslookup nginx-resolver-service.default.svc.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx-resolver-service.default.svc.cluster.local
Address 1: 10.98.47.5 nginx-resolver-service.default.svc.cluster.local


# to get ip 
controlplane $ k get pods -o wide 
NAME             READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
nginx-resolver   1/1     Running   0          14m     192.168.1.5   node01   <none>           <none>
super-pod        1/1     Running   0          34m     192.168.1.4   node01   <none>           <none>
testdns          1/1     Running   0          2m30s   192.168.1.6   node01   <none>           <none>

# to testdns using ip of svc
controlplane $ k exec -it testdns -- nslookup 192.168.1.5           
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      192.168.1.5
Address 1: 192.168.1.5 192-168-1-5.nginx-resolver-service.default.svc.cluster.local

#saving it to files

controlplane $ k exec -it testdns -- nslookup 192.168.1.5 > nginx.pod
controlplane $ k exec -it testdns -- nslookup nginx-resolver-service.default.svc.cluster.local > nginx.svc
controlplane $ ls
filesystem  nginx.pod  nginx.svc  q4.yaml  snap

```

### 6.There is a multi-container Deployment in the Namespace management which seems to have issues and is not getting ready.  write the logs of all containers to /root/logs.log 

```bash
#to see the deployments in management
kubectl get deploy -n management #management is namespace

# to edit multicontainer pod and it's deployment
kubectl edit deploy collect-data -n management

```

below questions are from this youtube video.
https://www.youtube.com/watch?v=vVIcyFH20qU 

### 7. Given an existing Kubernetes cluster running version 1.26.0, upgrade the master node and worker node to version 1.27.0. Ber sure to drain the master and worker node before upgrading it and uncordon it after the upgrade.(weightage 12)

search for upgrade and chose the correct version to upgrade.

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/


```bash
# drain the controlplane node
kubectl drain controlplane --ignore-daemonsets

#kubectl get nodes.  it will show the controlplane is drained and scheduling disabled.

ssh controlplane #get into controplane.

#1.upgrade kubeadm
# replace x in 1.27.x-* with the latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm='1.27.x-*' && \
apt-mark hold kubeadm

# replace x with the patch version you picked for this upgrade
sudo kubeadm upgrade apply v1.27.x


#Once the command finishes you should see:

[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.27.x". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgr

#Upgrade the kubelet and kubectl:

# replace x in 1.27.x-* with the latest patch version
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet='1.27.x-*' kubectl='1.27.x-*' && \
apt-mark hold kubelet kubectl

#restart the kubelet service

sudo systemctl daemon-reload
sudo systemctl restart kubelet

#exit to the first machine. now we can see our controlplane is upgraded
#needed to make schedulable again. 
kubectl uncordon controlplane


# now we need to upgrade the worker node

kubectl drain node01 --ignore-deamonsets #drain the node

#then ssh into the node machine.
#do the things same as the previous version

#Same as the first control plane node but use:

sudo kubeadm upgrade node
#instead of:

sudo kubeadm upgrade apply

#exit the node and uncordon the node01
```

### 8. Create a snapshot of ETCD and save it to /root/backup/etcd-backup-new.db. you can us the below certificates for takin the snapshot. CA certificate: /root/certificates/ca.crt Client certificate: /root/certificates/server.crt KEY: /root/certificates/server.key.  restore an old snapshot located at /root/backup/etcd-backup-old.db to /var/lib/etcd-backup.

search documentation for etcd snapshot

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

```bash
# for taking snapshot and saving it as new backup
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/root/certificates/ca.crt --cert=/root/certificates/server.crt --key=/root/certificates/server.key \
  snapshot save /root/backup/etcd-backup-new.db

#to backup the cluster

etcdutl --data-dir <data-dir-location> snapshot restore snapshot.db
#data -dir location is /var/lib/etcd-backup

#then go to /etc/kubernetes/manifests/etcd.yaml to change the location of etcd.
#change the hostpath named etcd-data to /var/lib/etcd-backup (directory where etcd backup happened)
```

### 9. Please join node01 worker node to the cluster, and you have to deploy a pod in the node01, pod name should be web and image should be nginx. (weightage 6)

search for token join in the documentation. we use this token to join the cluster.

https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-token/

```bash

# to create a token first we need to ssh into master node (controlplane)
ssh controlplane
#thern 
kubeadm token create --print-join-command #adding print join will help you to give a command to join the token. we will get a joining token
#then go to the using ssh and run the token.

#if you come across any issues run systemctl status kubelet
#restart the kubelet

# pod in the node
kubectl run web --image=nginx --dry-run=client -o yaml > pod-definition.yaml

kubectl apply -f pod-definition.yaml
```


### 10 please deploy a pod on node01 as per the below specifiction. Pod Name: web-pod, Container name = web, image: nginx. weightage(6). 6mean intermediate question.

it's a trouble shooting question so there will be an issue. 

```bash
kubectl run web --image=nginx --dry-run=client -o yaml > pod-definition.yaml

kubectl apply -f pod-definition.yaml

# the pods will be in pending state due to node01 is not ready.
# get inside the node. 
ssh node01
#check the status of kubelet service
systemctl status kubelet
#if kubelet is not working we need to restart the service.
sysetemctl start kubelet
# if restart is not working check the configuration settings
# the kubelet is not restarting is depends upon wrong configuration settings
# go to the path of the configuration of kubelet
# check all the paths in systemctl status kubelet
# kubelet is in /usr/bin folder
# then restart the service
systemctl daemon-reload

systemctl start kubelet
```

### 11. Mark the worker node named kworker as unschedulable and reschedule all the pods running on it (weightage 6)

https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

```bash
kubectl cordon <node name> #stopping further scheduling
kubectl drain --ignore-daemonsets <node name> #to drain the nodes
```


below questions are from this youtube video.
https://www.youtube.com/watch?v=4hxt-7lR3Tg


### 12 Create a new PersisentVolume named web-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /vol/data and no storageClassName defined. Next create a new PersistentVolumeClaim in Namespace production named web-pvc. it should not define a storageClassName. The PVC should bound to the PV correctly. Finally create a new Deployment web-deploy in Namespace production which mounts that volume at /tmp/web-data. The Pods of that Deployment should be of image nginx:1.14.2 (weightage 6).

https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/ this is the correct link for solving the question

pv.yaml
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: web-pv
  labels:
    type: local
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/vol/data"
```
```bash
kubectl apply -f pv.yaml #apply the yaml file

kubectl get namespace #to see namespaces

kubectl create ns production #to create namespace

kubectl get po #for pods
```

Now persistent volume claim
pvc.yaml
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: web-pvc
  namespace: production #for running in specific namespace
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

```bash
kubectl apply -f pvc.yaml #applying the yaml file
```

search deployments in documentaion and go to the below link.

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/


nginx-deployment.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  namespace: production
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: task-pv-storage
        persistentVolumeClaim:
          claimName: web-pvc
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: "/tmp/web-data"
          name: task-pv-storage
```
after this search for pv and you will get sample for mounting the volume

https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

```bash
kubectl apply -f nginx-deployment.yaml`

kubectl get deploy -n production

kubectl get pvc -n production #to see if it bounded or not. Bound option will be their
```

### 13 expose a existing pod called nginxpod, service name as nginxnodeportsvc, service should access through Nodeport (weightabe 6). Nodeport=30200

```bash
kubectl get svc #get the port by running 80

kubectl expose pod nginxpod --name=nginxnodeportsvc --port=80 --type=NodePort
#the service will be created

#now service will be exposed in a random port. we need to edit the port to expose in a certain port.

kubectl edit svc nginxnodeportsvc #we need to edit the service
#edit the nodeport field #then save and exist

kubectl get nodes -o wide #see the pod and it's ip. get the ip and port curl it.
```

### 14 Use Namespace project-1 for the following. Create a DaemonSet named daemon-imp with image httpd:2.4-alpine and labels id=daemon-imp and uuid=18426a0b-5f59-4e10-923f-c0e078e82462. The pods it creates should request 20 millicore cpu and 20 mebibyte memory. the pods of that DaemonSet should run on all nodes, also controlplanes. weightage(5).

Daemonset helps to run a particular pods on all the nodes. helps to deploy monitoring components.

search for daemonset in documentation

https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemon-imp
  namespace: project-1
  labels:
    id: daemon-imp
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
spec:
  selector:
    matchLabels:
      id: daemon-imp
      uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
  template:
    metadata:
      labels:
        id: daemon-imp
        uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: daemon-imp
        image: httpd:2.4-alpine
        resources:
          requests:
            cpu: 20m
            memory: 20Mi
```

```bash
#create the namespace
kubectl create namespace project-1

#applying the yaml file

kubectl apply -f <nameof yaml file>

kubectl get ds -n project-1
```

### 15 You have kubernetes cluster and running pods in multiple namespaces, The security team has mandated that the db pods on Project-a namespace only accessiable from the service pods that are running in Project-b namespace. weightage(4).

type network policies in the documentation. 
https://kubernetes.io/docs/concepts/services-networking/network-policies/

```bash
kubectl get po -o wide -n project-a #to get db and other important details

kubectl on project-b exec <podname> -- ping <db-ip> #to check connectivity of other pods to db #check it again to know the pod connection is correct or not

#we need to label the namespaces to get the details first.
kubectl get namespaces

kubectl label namespace project-a namespace=project-a

kubectl label namespace project-b namespace=project-b

kubectl get pods -n project-a --show-labels #for showing labels in each and every project.

#note all the labels. because network policies are working based on this labels.


```

np.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: project-a #we are doing this project on project-a namespace
spec:
  podSelector:
    matchLabels:
      app: db #label of db
  policyTypes:
  - Ingress #we don't need egress
  # - Egress
  ingress:
  - from:
    # - ipBlock: #no need of ip block
    #     cidr: 172.17.0.0/16
    #     except:
    #     - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          namespace: project-b #since we only need ingress from projct-b
      podSelector:  #remove the - in front of pod to make condition as and
        matchLabels:
          app: service #label of  service pod since we need connection from db pod to service pod

```

### 16. Create a new ServiceAccount gitops in Namesapce project-1. Create a Role and RoleBinding, both named gitops-role and gitops-rolebinding as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.

```bash
kubectl get ns #list the namespace

kubectl create sa gitops -n project-1 #to create service account named gitops in project-1 ns

#then we need to create the role

kubectl create role gitops-role --verb=create --resource=secrets,configmaps

kubectl describe role -n project-1 #to see created roles

#now we need to bind role and serviceaccount with rolebindings
kubectl create rolebinding gitops-rolebinding --role=gitops-role --serviceaccount=project-1:gitops

#--serviceaccount=namespace:serviceaccountname

k get rolebindin -n project-1 #to see the rolebindings

k -n project-1 auth can-i create pod --as system:serviceaccount:project-1:gitops #to check if you can create pod in space without creating resources

k -n project-1 auth can-i create configmap --as system:serviceaccount:project-1:gitops
```

### 17. There is a multi-container Deployment in Namespace management which seems to have issues and is not getting ready. Write the logs of all containers to /root/logs.log. Can you see the reason for failure.


```bash
kubectl get deploy -n management #getting deployment in management namespace
#we will see deployment named collect-data
kubectl -n management logs deploy/collect-data -c nginx >> /root/logs.log #to write the logs into logs file for nginx container
kubectl -n management logs deploy/collect-data -c httpd >> /root/logs.log #writing logs for httpd container
```

### 18 Deploy a pod called nginxpod with image nginx in controlplane, make sure pod is not scheduled in worker node. weightage(3).

```bash
k run --dry-run=client -o yaml > 18.yaml #it will save file into a yaml file
```

18.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginxpod
  name: nginxpod
spec:
  nodeName: controlplane #to specify the node edit the file in vi
  containers:
  - image: nginx
    name: nginxpod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### 19 expose a existing pod called nginxpod as a service, service name should be nginxsvc. Pod port=80 (weightage 4).

```bash
k expose po nginxpod --name=nginxsvc --port=80 #to expose the port
```

to check whether it worked or not.
```bash
controlplane $ k expose pod nginxpod --name=nginxsvc --port 80
service/nginxsvc exposed
controlplane $ k get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   27d
nginxsvc     ClusterIP   10.97.165.242   <none>        80/TCP    7s
controlplane $ curl 10.97.165.242:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
controlplane $ 
```

### 20. expose a existing pod called nginxpod, service name as nginxnodeportsvc, service should access through Nodeport. (weight 6). Nodeport = 30200

For defining a specific nodeport it's little tricky. Nodeport will be assigned randomly we need to assign a nodeport after it.

```bash
kubectl expose pods nginxpod --name=nginxnodeportsvc --port=80 --type=NodePort

controlplane $ kubectl get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP        27d
nginxnodeportsvc   NodePort    10.104.246.62   <none>        80:32480/TCP   10s #we can see port is assigned to a random port edit the service to reassign it to specific port.


controlplane $ k edit svc nginxnodeportsvc #edit the nodeport in to 30200
service/nginxnodeportsvc edited
#it will automatically redeployed
controlplane $ kubectl get svc
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes         ClusterIP   10.96.0.1       <none>        443/TCP        27d
nginxnodeportsvc   NodePort    10.104.246.62   <none>        80:30200/TCP   113s



controlplane $ k get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
controlplane   Ready    control-plane   27d   v1.30.0   172.30.1.2    <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.7.13
node01         Ready    <none>          27d   v1.30.0   172.30.2.2    <none>        Ubuntu 20.04.5 LTS   5.4.0-131-generic   containerd://1.7.13

#since it's node port the service is available at this port
controlplane $ curl 172.30.2.2:30200
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### 21. You can find an existing deployment frontenc in production namespace, scale down the replicas to 2 and change the image to nginx:1.25 weigtage(4).

Two ways to solve this problem. 1st edit the deployment yaml or using imperative commands

```bash
kubectl get deployment -n production #to get the deployments

kubectl edit deploy frontend -n production #to edit the deployment. 
#then edit the replicas and image
# It will get automatically deployed.

kubectl describe deploy frontend -n production #to see details after production
#you can see details of current replicas and deployments and logs of scale out
```

Imperative way to solve
```bash
kubectl scale deploy frontend --replicas=2 -n production #to scale down the replicas

kubectl set image deploy frontend nginx=nginx:1.25 -n production #to set the image name for nginx container and namespace production.

```

### 22. Auto scale the existing deployment frontend in production namespace at 80% of pod CPU usage, and set Minimum replicas=3 and Maximum replicas=5. (weightage 6).

Type horizontal autoscaling

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

https://kub[e](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)rnetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

```bash
kubectl -n production autoscale deploy frontend --min=3 --max=5 --cpu-percent=80

kubectl get hpa -n production #to get hpa 
```

### 23. Exposet existing deployment in production namespace named as frontenc throught Nodeport and Nodeport service name should be frontendsvc. weightage 4

Earlier we exposed pod here we need expose deployment.

```bash
k get deploy -n production #to see deployment

k -n production expose deploy frontend --name=frontendsvc --port=80 --type=NodePort

k get nodes -o wide #to see nodes
#then check the ip and port to see if it's works
```
### Troubleshooting.

### 24. You can find a pod named task-pv-pod in the default namespace, please check the status of the pod and troubleshoot, you can recreate the pod if you want. (weightage 3).

```bash

k describe pods task-pv-pod #see pod issues
#the issue was typo in pvc name. correct it and create it again
# there will be a temporary file will be created
# then delete the pods
# use temp file to create
k apply -f <tmp location?
```

### 25. Deply a pod with the following specifications:. (Pod Name: web-pod, Image: httpd, Node: Node01) Note: do not modify any settings on master and worker nodes. (weightage 6)

```bash
k run web-pod --image=httpd --dry-run=client -o yaml > 25.yaml
k describe pods web-pod
```

The pod is in pending state. there is lot of reason for a pod is being in pending state.
The issue was taints in node01.

```bash
controlplane $ k describe nodes controlplane | grep Taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
controlplane $ 
```
now we need to use tolerations for deploying pod in that specific node
search for taint in the documentation.
https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/


```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web-pod
  name: web-pod
spec:
  containers:
  - image: httpd
    name: web-pod
    resources: {}
  tolerations:
  - key: "node-role.kubernetes.io/control-plane" #we took the key from describe command
    operator: "Exists"
    effect: "NoSchedule"
  
status: {}
```

answer 

```bash
controlplane $ kubectl apply -f 25.yaml 
pod/web-pod created
controlplane $ kubectl get pods -o wide
NAME                                READY   STATUS              RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
nginx                               1/1     Running             0          50m   192.168.1.4   node01         <none>           <none>
nginx-deployment-77d8468669-mmqkn   1/1     Running             0          49m   192.168.1.5   node01         <none>           <none>
nginx-deployment-77d8468669-tfxj6   1/1     Running             0          49m   192.168.1.6   node01         <none>           <none>
nginx-deployment-77d8468669-v6q9g   1/1     Running             0          49m   192.168.1.7   node01         <none>           <none>
web-pod                             0/1     ContainerCreating   0          5s    <none>        controlplane   <none>           <none>
controlplane $ 
```


### 26. Create a new Persistent Volume named web-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /vol/data and no storageClassname defined. Next Create a new PersistentVolumeClaim in Namespace production named web-pvc. It should request 2Gi storage, accessMode ReadWrtieOnce and should not define a storageClassName. The PVC should bound to the PV correctly. Finally create a new deployment web-deploy in Namespace production which mounts that volume at /tmp/web-data. The Pods of that Deploymnet should be of image nginx:1.14.2 (weightage 6)

For createing pv 
https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

1pv.yaml
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: web-pv
  labels:
    type: local
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/vol/data"

```

```bash
controlplane $ kubectl apply -f 1pv.yaml 
persistentvolume/web-pv created
controlplane $ kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
web-pv   2Gi        RWO            Retain           Available                          <unset>                          5s

#to create production namespace
controlplane $ k create namespace production
namespace/production created
controlplane $ kubectl get namespace
NAME                 STATUS   AGE
default              Active   28d
kube-node-lease      Active   28d
kube-public          Active   28d
kube-system          Active   28d
local-path-storage   Active   28d
production           Active   7s
```


1pvc.yaml
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: web-pvc
  namespace: production #to create in specific namespace
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

search for deploy

deploy.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
  namespace: production
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: "/tmp/web-data"
          name: mypd
      volumes:
      - name: mypd
        persistentVolumeClaim:
          claimName: web-pvc
```

answer
```bash

controlplane $ kubectl get deployments.apps -n production 
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-deploy   3/3     3            3           84s

controlplane $ k get pvc -n production
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
web-pvc   Bound    pvc-012d5633-73d3-402d-a7a3-08f35d60a64a   2Gi        RWO            local-path     <unset>                 13m

controlplane $ k get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pvc-012d5633-73d3-402d-a7a3-08f35d60a64a   2Gi        RWO            Delete           Bound       production/web-pvc   local-path     <unset>                          84s
web-pv                                     2Gi        RWO            Retain           Available                                       <unset>                          16m
controlplane $ 
```


### 27. Create a kubernets Pod named "my-busybox" with the busybox:1.31.1 image. The Pod should run a sleep command for 4800 seconds. Verify that the Pod is running in Node01. (2)


```bash
controlplane $ k run my-busybox --image busybox:1.31.1 --command sleep 4800
pod/my-busybox created
controlplane $ k get pods
NAME         READY   STATUS    RESTARTS   AGE
my-busybox   1/1     Running   0          5s
controlplane $ k get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
my-busybox   1/1     Running   0          23s   192.168.1.4   node01   <none>           <none>
controlplane $ 
```

```bash
#in case it didn't work
k get nodes #and check the status of nodes
#cordon the node if it's not schedulable
```

https://kubernetes.io/docs/concepts/architecture/nodes/

### 28. You have a kubernets cluster that run a three-tier web application: a frontend tier(port 80), an application tier(port 8080), and a backend tier(3306). The security team has mandated that the backendtier should only be accessible from the application tier. (weight 4).

it means we need to implement a network policy.

install telnet 
```bash
k exec <pod name> -- sh -c "apt-get update"
k exec <pod name> -- sh -c "apt-get install -y telnet"

k exec <pod-name> telnet <ip-addr> <port> #see if you can connect from one pod to another in this.

# try it with all the pos one by one

k get po --show-labels #the network policies are working using this labels
```

search for network policies in the documentation

https://kubernetes.io/docs/concepts/services-networking/network-policies/

network-policy.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      tier: backend #label of backend application
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: application #label of frontend application
    ports:
    - protocol: TCP
      port: 3306 #port of backend
```
above file is only going to allow application with tier: application to port 3306 port to the pods with tier: backend label


After this check the connectivity to check if it's worked or not.

### 29. You have a kubernetes cluster and running pods in multiple namespaces, The security team has mandated that the db pods on Project-a namespace only accessible from the service pods that are running in Project-b namespace. (weightage 4).

```bash
k -n project-b exec <pod-namefrom> -- ping <ip of machine to check>
#it will check the connetivity of one pod from another pod.
```

Go to network policies and copy the yaml file

np.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: project-a
spec:
  podSelector:
    matchLabels:
      app: db #label of db pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          namespace: project-b #label of project-b
      podSelector:
        matchLabels:
          app: service #label of service pod
    #above condition is and condition. need correct both the condition

    #below condition is or condition. any one of condition will trigger it.
    - namespaceSelector:
      matchLabels:
          namespace: project-b #label of project-b
    - podSelector:
        matchLabels:
          app: service #label of service pod
```

inorder to implement this scenario we need to label the namespaces.

```bash
kubectl label namespace project-a namespace=project-a

kubectl get pods -n project-a --show-labels #to get the labesl of pods and work on it.
#add labels in all the namespaces

k -n project-b exec <pod-namefrom> -- ping <ip of machine to check>
#check the connectivity after this.
```


### 30. You can find a pod named multi-pod is running in the cluster and that is logging to a volume. You need to insert a sidecar container into the pod that will also read the logs from the volume using this command "tail -f /var/busybox/log/*.log". side specifications give below.

image: busybox:1.28
Name: sidecar
volumepath: /var/busybox/log

```bash
k get po multi-pod -o yaml > 1.yaml #to get pod configuration into a file.
cp 1.yaml 1copy.yaml #to copying details and have backup

kubectl logs multi-pod -c sidecar #to get logs of sidecar container.
```

add images and mount path accordingly
also add command: ["sh","-c","tail -f /var/busybox/log/*.log"]

### 31.Create a CronJob for running every 2 minuts with busybox image. The job name should be my-job and it should print the current date and time to the console. After running the job save any one of the pod logs to below path /root/logs.txt (2).


search for cronjob in documentation

https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

cronjob.yaml
```bash
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date;
          restartPolicy: OnFailure
```

```bash
k apply -f 13.yaml #cronjob is created it will work in 2 minutes interval

k get cronjob #to get cronjob

k get po --watch #to see pods getting created and working

k logs <cronjob name> > /root/logs.txt #to save it

```

### 32. Find the schedulable nodes in the cluster and save the name and count in to the below file. file path: /root/nodes.txt (1)

```bash
kubectl get nodes #to see the node

k get nodes -o jsonpath='{.items[*].spec.taints}' #to see the taints of nodes.

```
just save it and exist

### 33.please deploy a pod on Node01 as per the below specification(6)

pod name: web-pod
Container name = web
image: nginx

```bash
kubectl run web-pod --image=nginx --dry-run=client -o yaml > pod.yaml
#then edit the file and save it change the container name to web

k get nodes #check if the nodes are working
ssh node01 #get into the node which is not working
systemctl start kubelet #in case if it don't work restart
#still it doesn't work search the configuration files.

#search the paths
# then edit the configuration file
```

### 34. Please join node01 worker node to the cluster, and you have to deploy a pod in the node01, pod name should be web and image should be nginx. (weightage 6)

Search for token in the k8s documentation. You will get token command to run in the documentation.

search for kubeadm token or token join

https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-token/

```bash
k get nodes #to check available nodes

#we need to run a command to join node into our cluster
# we will be outside from the cluster in the exam. we need ssh into the master to run this command

ssh controlplane

kubeadm token create --print-join-command # we will get a joining token
# run this command in the node01. 
# the node will join the cluster
# restart the kubelet if you face issue
systemctl start kubelet

# tehn run 
k get nodes #to check whether it connected or not.

k run web --image=nginx
```

### 35. There was a security incident where an intruder was able to access the whole cluster from a single hacked web pod. To prevent this create a NetworkPolicy in default Namespace. It should allow the web-* pods only to: connect to service-* Pods on port 8080. After Implementation, connections from web-* Pods to application-* Pods on port 80 should also be blocked. (weightage 6)


```bash
k get pd #to get pods
k exec <web-pod-name> --telnet <ip-addr-pods> <portnumber> #to connection from webpod to other pds

k get po --show-labels #to see the labels of pod #we need labels for network policies

k apply -f network-policy.yaml #to create network policy using below pods

k get networkpolicy #to check policies

#then check the connectivity after this

```

network-policy.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpolicy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web #since we are controlling network out of web pods
  policyTypes:
  - Egress #we only need egress rules
  egress:
  - to:
    - podSelector:
       matchLabels:
          app: service  #only allowing connection to service from web
    ports:
    - protocol: TCP
      port: 8080
```

### 36. Create a new ServiceAccount gitops in Namespace project-1. Create a Role and RoleBinding, both named gitops-role and gitops-rolebinding as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace. (5% weightage)


```bash
k get ns #to check namespaces. Create it if it's not present

k create sa gitops -n project-1 #create service account named gitops in project-1 namespace

k get sa -n project-1 #to get service account in the project-1 namespace

k create role gitops-role --verb=create --resource=secrets,configmaps #allowing only to creat secrets,configmaps in resource namespace.

k get role -n project-1 #to see the roles

k describe role -n project-1 #to see details of roles. It will help you identify any mistakes.

# now we need to create rolebindings to connecting it with roles.

k create rolebinding -n project-1 gitops-rolebinding --role=gitops-role --serviceaccount=project-1:gitops

#--serviceaccount=namespace:serviceaccount name

k get rolebinding -n project-1 #to see the roles

#to check if it's working or not

k -n project-1 auth can-i create pod --as system:servcieaccount:project-1:gitops #permission to create pod in project-1 namespace

k -n project-1 auth can-i create configmap --as system:servcieaccount:project-1:gitops #creating configmap in project-1 namespace
```

### 37. There are two existing Deployments in Namespace world which should be made accessible via an Ingress. First: create clusterIP services for both Deployments for port 80. The Service should have the same name as the Deployments.

```bash
k get deploy -n world

k expose deply asia -n world --port=80 #to create service for deployment

cat /etc/hosts #to see the host entry in the namespace

k get ingressclass #to see ingress class name

k get ingress -n world #to check ingress rules

curl <ip address> #to see if you are able see the address

```

go to documentation and search for ingress. 
https://kubernetes.io/docs/concepts/services-networking/ingress/ 

ingress.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: world
  namespace: world #ingress resources are name spaced so we need specify those names.
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx #by running k get ingressclass
  rules:
  - host: "world.universe.mine" #copy and paste the domain name
    http:
      paths:
      - path: /europe #path should be this
        pathType: Prefix
        backend:
          service:
            name: europe #the service should be europe
            port:
              number: 80
      - path: /asia #path should be this
        pathType: Prefix
        backend:
          service:
            name: asia #the service should be europe
            port:
              number: 80
```

### 38. Use Namespace project-1 for the following. Create a DaemonSet named daemon-imp wiht image httpd:2.4-alpine and labels id=daemon-imp and uuid=18426a0b-5f59-4e10-923f-c0e078e82462. The Pods it creates should request 20 millicore cpu and 20 mebibyte memory. The Pods of that DaemonSet should run on all nodes, also controlplanes (weightage = 5%)

search for daemonset in k8s documentation.
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/


daemonset.yaml
```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemon-imp
  namespace: project-1
  labels:
    id: daemon-imp
    uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
spec:
  selector:
    matchLabels:
      id: daemon-imp
      uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
  template:
    metadata:
      labels:
        id: daemon-imp
        uuid: 18426a0b-5f59-4e10-923f-c0e078e82462
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: daemon-imp #not mentioned
        image: httpd:2.4-alpine
        resources:
          requests:
            cpu: 20m
            memory: 20Mi
```

```bash
k apply <nameoffile>.yaml

k create ns project-1

k get ds -n project-`
```

### 39. ETCD backup

search for etcd snapshot in documentation

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

```bash
cat /etc/kubernets/manifest/etcd.yaml #find the details 

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>  #fill this


etcdutl --data-dir <data-dir-location> snapshot restore snapshot.db #to take backup
```

### 40. Create a snapshot of ETCD and save it to /root/backup/etcd-backup-new.db. you can use the below certificates for taking the snapshot. (CA certificate: /root/certificates/ca.crt, Client certificate: /root/certificates/server.crt, key: /root/certificates/server.key). restore an old snapshot located at /root/backup. 

 Like the eralier questions.


### 41. You can find a pod named multi-container-pod running in the cluster, take the cont