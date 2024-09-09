# previous year questionaires and answers.

https://www.youtube.com/watch?v=o_7jlMBHFFA

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
kubectl run web-pod --image=nginx --dry-run=client -o yaml > pod-definition.yaml
vi pod-definition.yaml #then change the name to web

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

k -n project-b exec <pod-name> -- ping <ip-of-pod>


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

kubectl create role gitops-role --verb=create --resource=secrets,configmaps -n project-1

kubectl describe role -n project-1 #to see created roles

#now we need to bind role and serviceaccount with rolebindings
kubectl create rolebinding gitops-rolebinding --role=gitops-role --serviceaccount=project-1:gitops -n project-1

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

### 23. Expose the existing deployment in production namespace named as frontenc throught Nodeport and Nodeport service name should be frontendsvc. weightage 4

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
    #above condition is 'and' condition. need correct both the condition

    #below condition is 'or' condition. any one of condition will trigger it.
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

https://www.youtube.com/watch?v=Zm5sy6otLGc 

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

```bash
spec:
  containers:
  - image: nginx:latest
    name: web-pod
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /var/log/nginx
      name: hostpath-volume #same volume in both the containers
  - image: busybox:1.28
    name: sidecar
    command: ['sh','-c','tail -f /var/busybox/log/*.log'] #it can access it easily
    volumeMounts:
    - mountPath: /var/busybox/log
      name: hostpath-volume #same volume in both the containers
  volumes:
  - hostPath:
      path: /var/volume
    name: hostpath-volume


```

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

#check the paths

k run web --image=nginx

vi /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf #edit the configuration file
```


```bash

node01 $ systemctl status kubelet.service 
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Sun 2024-09-08 18:17:48 UTC; 37min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 1123 (kubelet)
      Tasks: 10 (limit: 2338)
     Memory: 41.5M
     CGroup: /system.slice/kubelet.service
             └─1123 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/k

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


### 41. You can find a pod named multi-container-pod running in the cluster, take the container logs and the container id of the c2 container and save it into the below mentioned location. Restart the C2 container and write the cluster events to the /root/event.log file. Log save to /root/logs.txt, Container id to /root/id.txt (4).

```bash
k get pods 

k edit multi-container-pod #to see the container name

k logs multi-container-pod -c c2 >> /root/log.txt #to storing logs into a file.

#to run the container id login into node01. 
crictl ps #since docker commands won't work. you will get container id's. copy the respective one and save it in /root/id.txt file.

#for saving events after restarting c2 containers.
#go to node01
crictl stop <container-id>
crictl rm <container-id>

#go back 

kubectl get events --sort-by=.metadata.creationTimestamp > /root/logs.txt

#to get event logs of pods #if they ask about events of pods instead of whole cluster.
kubectl get events --field-selector involvedObject.name=multi-container-pod
```

run kubectl quick reference/ kubectl cheat sheet

https://kubernetes.io/docs/reference/kubectl/quick-reference/

search for events in it

### 42. Find the Pod with the highest priority in Namespace management and delete it.

```bash
k get po -n management #to list pods in management cluster.

k edit po -n management <pod-name> #to find the priorities

#there will be priority and priority classname
# highest number in priority and priority classname can be deleted.

k get priorityclass #to see the prioriyclass

k -n management get pod -o yaml | grep -i priority -B 20 #to get priority class with 20 lines before priority. this is the right way of doing it.

k delete po <pod-name> -n management #delete the second pod
```

### 43. In Namespace lion there is one existing pod which requests 1Gi of memory resources. That Pod has a specific priority because of its PriorityClass. Create new Pod named important of image nginx:1.21.6-alpine in the Namespace. It should request 1Gi memory resources. Assign a higher priority to the new Pod so it's scheduled instead of the existing one. Both pods won't fit in the cluster.


```bash
 k get po -n lion #to see pods

k edit pod <pod-name> -n lion #to see the priority and priority class

k run important -n lion --image=nginx:1.21.6-alpine --dry-run=client -o yaml > 43.yaml
#to create the important file

k get priorityclasses #to see the important priority classes.
k edit po <pod-name> -n lion #edit the priorityclasses


```

https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/  search for priority add `priorityClassName:`

search for resource and request
https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```

```bash
    resources:
      requests:
        memory: "64Mi" #this request should be 1Gi
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

To create a priorityclass. Search for priorityclass in documentation.

pc.yaml
```bash
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: level4
value: 4000000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```
to create priorityclass `kubectl apply -f pc.yaml`


### 44. Create a replicaset with below specifications. (Name=web-app, Image=nginx, Replicas=3). Please note, there is already a pod running in our cluster named web-frontend, please make sure the total number of pods running in the cluster is not more than 3. (weightage 4).

```bash

#to add replicaset to already exisiting pods.
kubectl get po #to get details of pod
kubectl get po --show-labels #use this label to create the appication

kubect edit po frontend #to see the labels. We need those labels to add pod to the replicaset.
# there is one label named `tier: frontend`.

k create deploy web-app --image=nginx --dry-run=client -o yaml > replicaset.yaml
#creating deploy file helps to create replicasets easily. Since we need to just change the kind.

k get pods #to see the replicasets
```

replicaset.yaml
```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  creationTimestamp: null
  labels:
    tier: frontend #label from the existing application.
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  # startegy: {} #we don't want startegy in replicaset
  template:
    metadata:
      creationTimestamp: null
      labels:
        tier: frontend
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

### 45. Create a ConfigMap named trauerweide with content tree=trauerweide. Create the ConfigMap stored in existing file /root/cm.yaml


```bash
controlplane $ kubectl create configmap trauerweide --from-literal=tree=trauerweide
configmap/trauerweide created
controlplane $ k get cm
NAME               DATA   AGE
kube-root-ca.crt   1      30d
trauerweide        1      9s
controlplane $ ls
cm.yaml  filesystem  snap
controlplane $ kubectl create -f cm.yaml 
configmap/birke created
controlplane $ k get cm\
> ^C
controlplane $ k get cm 
NAME               DATA   AGE
birke              3      7s
kube-root-ca.crt   1      30d
trauerweide        1      30s
controlplane $ 
```

### 46. Create a Pod named pod1 of image nginx:alpine. Make key tree of ConfigMap trauerweide available as environment variable TREE1, Mount all keys of ConfigMap birke as volume. The files should be available under /etc/birke/*, Test env+volume access in the running Pod

search for configmaps in the documentation
https://kubernetes.io/docs/concepts/configuration/configmap/ 

pod.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: nginx:alpine
    name: pod1
    env:
        # Define the environment variable
        - name: TREE1 # Notice that the case is different here
                        # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: trauerweide         # The ConfigMap this value comes from.
              key: tree
    volumeMounts:
    - name: birke
      mountPath: "/etc/birke"
  volumes:
  # You set volumes at the Pod level, then mount them into containers inside that Pod
  - name: birke
    configMap:
      # Provide the name of the ConfigMap you want to mount.
      name: birke
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

```bash
controlplane $ k run pod1 --image=nginx:alpine --dry-run=client -o yaml > pod.yaml
controlplane $ ls
cm.yaml  filesystem  pod.yaml  snap

k apply -f pod.yaml

#to check everything
controlplane $ kubectl exec pod1 -- env | grep "TREE1=trauerweide"
TREE1=trauerweide
controlplane $ kubectl exec pod1 -- cat /etc/birke/tree
birke
controlplane $ kubectl exec pod1 -- cat /etc/birke/level
3
controlplane $ kubectl exec pod1 -- cat /etc/birke/department
park
```

### 47. List the pods in the safari Namespace, sorted by creation time and save the command to the below path. /root/pods_timestamp.txt (weight 2).

```bash
k get po -n safari

k get po -n safari --sort-by=.metadata.creationTimestamp
#to sort by creation timestamp

k get po -n safari --sort-by=.metadata.creationTimestamp | tac #for ascending order

#copy and save the command in the file
#mention in full name kubectl

kubectl get pods -n safari --sort-by=.spec.priority #to sort it by priority
```

### 48. Create a new deployment named 'web' using the 'nginx:1.16' image with 3 replicas. Ensure that no pods are scheduled on the node named 'kworker'. (weightage 4).

```bash
#use the cordon command to make the node unschedulable
kubectl cordon kworker

kubectl create deploy web --image=nginx:1.16 --dry-run=client -o yaml > deploy.yaml

#then edit the deployement replicas

k apply -f deploy.yaml

k get po -o wide #to check the pods

kubectl uncordon kworker #to enable scheduling
```

deploy.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
        resources: {}
status: {}
```

`kubectl apply -f deploy.yaml`

### 48. Mark the worker node named kworker as unschedulable and reschedule all the pods running on it. (weightage 6)

```bash
kubectl get pods -o wide

kubectl cordon kworker

kubectl drain --ignore-daemonsets kworker
```


### 49. Given an existing kubernetes cluster running version 1.26.0, upgrade the master node and worker node to version 1.27.0. Be sure to drain the master and worker node before upgrading it and uncordon it after the upgrade. weightage = 12%.

search for upgrade in k8s documentation. Read the below documentation
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/ 


must asking and practice it. most weightage question

### 50. Add an init container named init-container (which has been defined in  spec file /home/master/opt/web-pod.yaml). The init container should create an empty file named /workdir/conf.txt. If /workdire/conf.txt is not detected, the pod should exit. Once the spec file has been updated with the init container definition, the pod should be created. (weightage = 4%)

search for init in documentation

https://kubernetes.io/docs/concepts/workloads/pods/init-containers/ 

init.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  volumes:
  - name: workdir
    emptyDir:
  containers:
  - name: web-pod
    image: alpine
    command: ['sh', '-c', 'if [-f /workdir/conf.txt]; then sleep 10000; else exit 1; fi']
    volumentMounts:
    - name: workdir
      mountPath: /workdir
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'touch /workdir/conf.txt']
    volumentMounts:
    - name: workdir
      mountPath: /workdir
```

### 51. Create a namespace finance, and Create a NetworkPolicy that blocks all traffic to pods in finance namespace, except for traffic from pods in the same namespace on port 8080.


```bash
kubectl create namespace finance #create the namespace

kubectl label namespace finance app=finance #to label the namespace

kubectl get namespace finance --show-labels #to show labels

kubectl create -f <filename>
```

go to documentaion and search for network policies

networkpolicy.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: finance
  namespace: finance
spec:
  podSelector: {} #all pods in the finance namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          app: finance 
    ports:
    - protocol: TCP
      port: 8080
```

### 52. Create a NetworkPolicy that denies all access to the payroll Pod in the accounting namespace

```bash

kubectl get pods -n accounting
kubectl get pods -n accounting --show-labels
kubectl create -f payroll-policy.yaml
```

go to documentation and search for network policy
https://kubernetes.io/docs/concepts/services-networking/network-policies/

payroll-policy.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payroll-policy
  namespace: accounting
spec:
  podSelector:
    matchLabels:
      app: payroll
  policyTypes:
  - Ingress
  - Egress
```

### 53. Install and configure a 3 nodes kubernetes cluster with kubeadam

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
search for installing kubeadm

https://www.youtube.com/watch?v=IQtrNRIW0uQ&list=PLkDIJDlFRu0xVERZaAfMXB6pg28FNH4Nk&index=4&pp=iAQB

```bash
lsb_release -a
```

### 54. Troubleshoot a kubernetes cluster. Explore the cluster, determine and fix the issue.


```bash
k get nodes
k describe nodes <node-name> #you will see kubelet stopped working

ssh <node-name> 

journalctl -u kubelet #to get logs of kubelet

sudo systemctl start kubelet #to restart the kubelet

sudo systemctl enable

sudo systemctl start kubelet

sudo systemctl status kubelet

```

### 55. Deployment named nginx-deployment is created in the default namespace, scale the deployment to 8 replicas.

```bash
kubectl scale deploy nginx-deployment --replicas=8
```

### 56.Create pod named multicontainer with multiple containers. redis + nginx


```bash
kubectl run multicontainer --image=nginx --dry-run=client -o yaml > 1.yaml

```

muliti.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multicontainer
  name: multicontainer
spec:
  containers:
  - image: nginx
    name: multicontainer
  - image: redis
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### 57. Make the node k8s-worker1 unschedulable and reschedule the pods running on it.

```bash
kubectl cordon <node-name> #to make unschedulable

kubectl drain <node-name> --ignore-daemonsets#to reschedule the nodes
```

### 58. Create a deployment named nginx-deploy in the default namespace using the image nginx:1.18, with 2 replicas. Perform a rolling deployment to update the image to nginx:1.19 and record this change. Scale the deployment to 4 replicas.

Search for deployment in documentation 

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

search documentation for record in deployments page

dp.yaml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  replicas: 2
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
        image: nginx:1.18
        ports:
        - containerPort: 80

```

```bash
controlplane $ k apply -f dp.yaml 
deployment.apps/nginx-deploy created
controlplane $ k get deployments.apps 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   0/2     2            0           6s
controlplane $ k get deployments.apps 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   2/2     2            2           11s

kubectl set image deployment/nginx-deploy nginx=nginx1.19 --record #for recording a change

kubectl rollout status deployment/nginx-deploy

kubectl sclae deployment nginx-deploy --replicas=4

kubectl get deploy


controlplane $ kubectl rollout history deployment nginx-deploy 
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/nginx-deploy nginx=nginx1.19 --record=true
```

### 59. Run a pod using the image redis and ensure the pod is run on each worker node.

search for daemonsets
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      name: redis
  template:
    metadata:
      labels:
        name: redis
    spec:
      containers:
      - name: redis
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
```

### 60. Create a static pod on the worker node k8s-worker1 using the image nginx

create a pod.yaml file for it and save it in /etc/kubernetes/manifests/pod.yaml inside the specific node

find the config file of kublet and find the static pod path. then create a pod.yaml file in it
```bash
ps -aux | grep kubelet
```

### 61. Create a new pod called web-pod with image busybox Allow the pod to be able to set system_time. The container should sleep for 3200 seconds. (7%)

We need to search for system capabilites in the documentation.
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

```bash
kubectl run web-pod --image=busybox --command sleep 3200 --dry-run=client -o yaml > pod.yaml #to create the file


```

pod.yaml
```bash
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: web-pod
  name: web-pod
spec:
  containers:
  - command:
    - sleep
    - "3200"
    image: busybox
    name: web-pod
    securityContext:
      capabilities:
        add: ["SYS_TIME"] #it's from searching time in security context file
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### 62. Create a new deployment called myproject, with image nginx:1.16 and ​1 replica. Next upgrade the deployment to version 1.17 using rolling​ update​. Make sure that the version upgrade is recorded in the resource annotation.​

```bash

controlplane $ k create deployment myproject --image nginx:1.16 --replicas 1
deployment.apps/myproject created

controlplane $ k set image deployment/myproject nginx=nginx1.17 --record
Flag --record has been deprecated, --record will be removed in the future
deployment.apps/myproject image updated

kubectl annotate #since --record is depreciated

#nginx=nginx1.17 nginx is container name. and set command will do a rolling update

#to check if it's worked or not
controlplane $ k describe deployments.apps myproject 
Name:                   myproject
Namespace:              default
CreationTimestamp:      Wed, 04 Sep 2024 17:00:49 +0000
Labels:                 app=myproject
Annotations:            deployment.kubernetes.io/revision: 2
                        kubernetes.io/change-cause: kubectl set image deployment/myproject nginx=nginx1.17 --record=true
Selector:               app=myproject
Replicas:               1 desired | 1 updated | 2 total | 1 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=myproject
  Containers:
   nginx:
    Image:         nginx1.17
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  myproject-6777d8b55f (1/1 replicas created)
NewReplicaSet:   myproject-77879bc788 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  3m7s  deployment-controller  Scaled up replica set myproject-6777d8b55f to 1
  Normal  ScalingReplicaSet  74s   deployment-controller  Scaled up replica set myproject-77879bc788 to 1



#to see the history
controlplane $ kubectl rollout history deployment myproject 
deployment.apps/myproject 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment/myproject nginx=nginx1.17 --record=true

```

### 63. Create a new deployment called my-deployment. Scale the deployment to 3 replicas.   Make sure desired number of pod always running. 

```bash
controlplane $ k create deployment my-deployment --image=nginx --replicas 3
deployment.apps/my-deployment created

controlplane $ k get deployments.apps 
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   3/3     3            3           15s
```

### 64. Deploy a web-nginx pod using the nginx:1.17 image with the labels set to tier=web-app.

```bash
controlplane $ k run web-nginx --image=nginx:1.17 --labels="tier=web-app"
pod/web-nginx created
```

### 67.  Create a static pod on node01 called static-pod with image nginx and you     have to make sure that it is recreated/restarted automatically in case ​of any failure happens

static pod is pod that associated with a default kubelet or pods.


```bash

node01 $ ps aux | grep kubelet #to find the config of kublet
root        1188  0.6  4.5 1927452 92920 ?       Ssl  16:38   0:15 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9 --container-runtime-endpoint unix:///run/containerd/containerd.sock --cgroup-driver=systemd --eviction-hard imagefs.available<5%,memory.available<100Mi,nodefs.available<5% --fail-swap-on=false


#--config=/var/lib/kubelet/config.yaml this is the default location

node01 $ cat /var/lib/kubelet/config.yaml | grep static
staticPodPath: /etc/kubernetes/manifests #this will show the static pod location

#create a pod yaml and place it inside this location it will become a static pod

# run this command and create a yaml file and copy that into static pod location
controlplane $ k run static-pod --image=nginx --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-pod
  name: static-pod
spec:
  containers:
  - image: nginx
    name: static-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### 68. Create a pod called pod-multi with two containers, as given below:​
Container 1 - name: container1, image: nginx​
Container2 - name: container2, image: busybox, command: sleep 4800​

```bash

#first create one container and add the second one

kubectl run pod-multi --image=nginx --dry-run=client -o yaml >> pod.yaml

#then add the second container
```

pod.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod-multi
  name: pod-multi
spec:
  containers:
  - image: nginx
    name: pod-multi
  - name: hello
    image: busybox
    command: ['sh', '-c', 'sleep 4800']
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```


### 69. Create a pod called test-pod in "custom" namespace belonging to the test environment (env=test) and backend tier (tier=backend).​ image: nginx:1.17​

```bash

kubectl get namespaces #to check whether the namespace is present or not.

kubectl create namespace custom #the namespace name should be custom
```

pod.yaml
```bash
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: test-pod
    env: test
    tier: backend
  name: test-pod
  namespace: custom
spec:
  containers:
  - image: nginx:1.17
    name: test-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

### 70.  Get the node node01 in JSON format and store it in a file at ​ ./node-info.json​

```bash
k get nodes

k get nodes node01 -o json > ./node-info.json #to get node details #can get from kubectl cheatsheet
```

### 71. Use JSON PATH query to retrieve the oslmages of all the nodes and store it in a file “all-nodes-os-info.txt” at root location.​ (Note: The osImage are under the nodeInfo section under status of each node.​)

go to k8s documentation under kubectl cheatsheet. You will see something like this
```bash

# Get ExternalIPs of all nodes
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# List Names of Pods that belong to Particular RC

kubectl get node -o json #to get all the details in json format

controlplane $ kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > all-node-os-info.txt
controlplane $ cat all-node-os-info.txt 
Ubuntu 20.04.5 LTS Ubuntu 20.04.5 LTScontrolplane $ 
```

### 72.Create a Persistent Volume with the given specification.​ Volume Name: pv-demo​ Storage:100Mi​ Access modes: ReadWriteMany​ Host Path: /pv/host-data​

search for pv in documentation

```bash
controlplane $ cat pv1.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/host-data

controlplane $ k apply -f pv1.yaml 
persistentvolume/pv-demo created
controlplane $ k get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-demo   100Mi      RWX            Retain           Available                          <unset>                          4s
```

### 73. Worker Node “node01” not responding, Debug the issue and fix it.​

```bash
#go to node01 by doing ssh 
ssh node01
#if we were unable to ssh then it's an issue with unreachability of network

#other issue will be kubelet not running
ps aux | grep kubelet

journactl -xe #to see the error in the kubectl

#to see the configuration of kubelet go to
ls /etc/kubernetes/kubelet.conf

#client certificat and client-key should be in lib directory.

    client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
    client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
#like above
# check the location

systemctl restart kubelet #to restart the kubelet

ps aux | grep kubelet #to check

```

### 74. Upgrade the Cluster (Master and worker Node) from 1.18.0 to 1.19.0. Make sure to first drain both Node and make it available after upgrade.​ (most important question)

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/


### 75. Take a backup of the ETCD database and save it to “/opt/etcd-backup.db” . Also restore the ETCD database from the backup​

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#securing-etcd-clusters

```bash

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>

k describe pods -n kube-system etcd-controlplane #to get the details of ca, cert, key

#or the easiest way
cat /etc/kuberenetes/manifests/etcd.yaml | grep file

#example command 
ETCDCTL_API=3 etcdctl --endpoints=https://172.30.1.2:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kube
rnetes/pki/etcd/server.key --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd-backup.db


#to check the snapshot
controlplane $ etcdctl --write-out=table snapshot status /opt/etcd-backup.db 
Deprecated: Use `etcdutl snapshot status` instead.

+---------+----------+------------+------------+
|  HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+---------+----------+------------+------------+
| 14f13e5 |     2870 |       2201 |     4.2 MB |
+---------+----------+------------+------------+


etcdutl --data-dir <data-dir-location> snapshot restore snapshot.db

etcdutl --data-dir /var/lib/etcd-backup snapshot restore /opt/etcd-backup.db
#then we need to go to the
# datadir is where we restore the data


cd /etc/kubernetes/manifests
vi etcd.yaml #to edit it after restoration

# then change the hostpath to /var/lib/etcd-backup that's where we restored the data
```

### 76. Create a new user “ajeet”. Grant him access to the cluster. User “ajeet” should have permission to create, list, get, update and delete pods. The private key exists at location:​ /root/ajeet/.key and csr at /root/ajeet.csr

search for csr in kubernetes documentation. You will get a page like this below
https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/


```bash
openssl genrsa -out /root/ajeet.key 2048 #to create private key

openssl req -new -key /root/ajeet.key -out /root/ajeet.csr -subj "/CN=ajeet"

controlplane $ ls
ajeet.csr  ajeet.key  filesystem  snap

#create a certificate signing request

cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF

#save the important part oof into a file. for request we need to give base64 encoded value for that. give

cat /root/ajeet.csr | base64 | tr -d "\n"

#copy the output and paste it in the request portion. we need copy till equals sign otherwise it will throw errors.

kubectl apply -f csr.yaml 
#apply it

controlplane $ k get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
ajeet       2m51s   kubernetes.io/kube-apiserver-client           kubernetes-admin           24h                 Pending


#it's in pending state we need to approve it

controlplane $ kubectl certificate approve ajeet
certificatesigningrequest.certificates.k8s.io/ajeet approved


controlplane $ k get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
ajeet       5m55s   kubernetes.io/kube-apiserver-client           kubernetes-admin           24h                 Approved,Issued
csr-5p484   34d     kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:omugk8    <none>              Approved,Issued
csr-vbx96   34d     kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   <none>              Approved,Issued

#now it's approved

#Retrieve the certificate from the CSR:

kubectl get csr/ajeet -o yaml
#The certificate value is in Base64-encoded format under status.certificate.

#Export the issued certificate from the CertificateSigningRequest.

kubectl get csr ajeet -o jsonpath='{.status.certificate}'| base64 -d > ajeet.crt


#now we need to create role and rolebindings
# This is a sample command to create a Role for this new user:
controlplane $ kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
role.rbac.authorization.k8s.io/developer created


#This is a sample command to create a RoleBinding for this new user:
controlplane $ kubectl create rolebinding developer-binding-ajeet --role=developer --user=ajeet
rolebinding.rbac.authorization.k8s.io/developer-binding-ajeet created


# Add to kubeconfig
# The last step is to add this user into the kubeconfig file.

# First, you need to add new credentials:

kubectl config set-credentials ajeet --client-key=ajeet.key --client-certificate=ajeet.crt --embed-certs=true
# Then, you need to add the context:

kubectl config set-context ajeet --cluster=kubernetes --user=ajeet
# To test it, change the context to myuser:

kubectl config use-context ajeet

controlplane $ kubectl config set-credentials ajeet --client-key=ajeet.key --client-certificate=ajeet.crt --embed-certs=true
User "ajeet" set.
controlplane $ kubectl config set-context ajeet --cluster=kubernetes --user=ajeet
Context "ajeet" created.
controlplane $ kubectl config use-context ajeet
Switched to context "ajeet".


controlplane $ kubectl config get-contexts 
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         ajeet                         kubernetes   ajeet              
          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   
          myuser                        kubernetes   ajeet


controlplane $ kubectl auth can-i list pods --as=ajeet
yes
controlplane $ kubectl auth can-i list rc --as=ajeet
no
controlplane $ kubectl auth can-i create pods --as=ajeet
yes
controlplane $ kubectl auth can-i delete pods --as=ajeet
yes
controlplane $ kubectl auth can-i get pods --as=ajeet
yes

```

### 77. (a) Create a Nginx pod dns-resolver using image nginx, (b) expose it internally with a service called dns-resolver-service. (c) Check if service name is resolvable from within the cluster.Use the image busybox:1.28 for dns lookup. (b) Save the result in /root/nginx.svc

Expose internally means using a cluster ip and create a service.

```bash

kubectl run dns-resolver --image=nginx #just run this since they asked for creating a pod

# for exposeing it internally. Pod is giving pod name
kubectl expose pod dns-resolver --name=dns-resolver-service --port=80 --target-port=80 --type=ClusterIP

#creating an image for dns lookup
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup dns-resolver-service

controlplane $ kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup dns-resolver-service
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      dns-resolver-service
Address 1: 10.96.55.158 dns-resolver-service.default.svc.cluster.local
pod "test-nslookup" deleted

# -- nslookup dns-resolver-service it's the command that needed to be run inside the container it will simply output the result and will not restarted again.

#to save the output to a file

kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup dns-resolver-service > /root/nginx.svc


controlplane $ k get pods -o wide 
NAME           READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
dns-resolver   1/1     Running   0          4m20s   192.168.1.5   node01   <none>           <none>
task-pv-pod    1/1     Running   0          28m     192.168.1.4   node01   <none>           <none>
controlplane $ kubectl run test-nslookup --image=busybox --rm -it --restart=Never -- nslookup 192.168.1.5         
Server:         10.96.0.10
Address:        10.96.0.10:53

5.1.168.192.in-addr.arpa        name = 192-168-1-5.dns-resolver-service.default.svc.cluster.local

pod "test-nslookup" deleted
```

### 78. A pod “appychip” (image=nginx) in default namespace is not running.​ Find the problem and fix it and make it running.

There was a taint in the node01 so we need to add toleration in the specific node so that it can be placed in the specific node

https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
search for taints and tolerations

example.yaml
```bash
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    value: "value1"
    effect: "NoSchedule"
```

### 79. Create a ReplicaSet (Name: appychip, Image: nginx:1.18, Replica: 4)​. There is already a Pod running in a cluster.​ Make sure that the total count of pods running in the cluster is not more than 4​

take the configuration of existing pod.
reate a replicaset to match the existing pod. 
For that, we would need to create a new replicaset definition file with matchLabels that must be matched the Pod's matchLabels. Just scaling the Replicaset would not be the right solution.

search for replicaset in documentation https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

example.yaml
```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  creationTimestamp: "2023-09-05T03:02:18Z"
  generation: 1
  labels:
    run: app-chip #label of currentl running pod
  name: appychip
spec:
  replicas: 4
  selector:
    matchLabels:
      run: app-chip
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: app-chip
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: app-chip
```

### 80. Create a Network Policy named "appychip" in default namespace​
There should be two types, ingress and egress. ​
The ingress should block traffic from an IP range of your choice except some other IP range. Should also have namespace and pod selector.​
Ports for ingress policy should be 6379​
For Egress, it should allow traffic to an IP range of your choice on 5978 port.​


example.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: appychip
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```


### 81. You have to access to multiple clusters from your main terminal through kubectl contexts. Write all those context names into */opt/course/1/contexts*. Next write a command to disply the current context into */opt/course/1/context_default_kubectl.sh*, the command should use *kubectl*. Finally write a second command doing the same things into */opt/course/1/context_default_no_kubectl.sh*, but without the use of kubectl.


```bash

kubectl config get-contexts -o name > /opt/course/1/contexts

controlplane $ touch /opt/course/1/contexts
controlplane $ kubectl config get-contexts -o name > /opt/course/1/contexts
controlplane $ cat /opt/course/1/contexts 
kubernetes-admin@kubernetes

#current context into file

controlplane $ vi /opt/course/1/context_default_kubectl.sh
controlplane $ chmod +x /opt/course/1/context_default_kubectl.sh 
controlplane $ ./context_default_kubectl.sh 
kubernetes-admin@kubernetes
controlplane $ cat context_default_kubectl.sh 
kubectl config current-context  #command to see current context

#without kubectl 
controlplane $ cat ~/.kube/config | grep current
current-context: kubernetes-admin@kubernetes

#save it in a file and run

controlplane $ echo "cat ~/.kube/config | grep current" > context_default_no_kubectl.sh
controlplane $ chmod +x context_default_no_kubectl.sh 
controlplane $ ./context_default_no_kubectl.sh 
current-context: kubernetes-admin@kubernetes
```

### 82. There are various Pods in all namespaces. Write a command into /opt/course/5/find_pods.sh which lists all Pods sorted by their AGE (metadata.creationTimestamp). Write a second command into /opt/course/5/find_pods_uid.sh which lists all Pods sorted by field metadata.uid. Use kubectl sorting for both commands.

this command is in kubectl cheatsheet. search for sort.

```bash
echo "kubectl get pods -A --sort-by=metadata.creationTimestamp" > /opt/course/5/find_pods.sh


echo "kubectl get pods -A --sort-by=metadata.uid" > /opt/course/5/find_pods_uid.sh
```


### 84. Ssh into the controlplane node with ssh cluster1-controlplane1. Check how the controlplane components kubelet, kube-apiserver, kubescheduler, kube-controller-manager and etcd are started/installed on the controplane node. Also find out the name of the DNS application and how it's started/installed on the controlplane node. 
write your findings into file /opt/course/8/controlplane-componets.txt. The file should be structured like:
 /opt/course/8/controlplane-componenet.txt
 kubelet:[type]
 kube-apiserver:[type]
 kube-scheduler:[type]
 kube-controller-manager:[type]
 etcd:[type]
 dns:[type]

choices of :[type] are : not-installed, process, static-pod, pod


```bash
controlplane $ find /etc/systemd/system/ | grep kube
/etc/systemd/system/multi-user.target.wants/kubelet.service  #kubelet is running as process
controlplane $ find /etc/systemd/system/ | grep etcd #etcd is not running as pd
controlplane $ find /etc/kubernetes/manifests/
/etc/kubernetes/manifests/
/etc/kubernetes/manifests/etcd.yaml
/etc/kubernetes/manifests/kube-scheduler.yaml
/etc/kubernetes/manifests/kube-controller-manager.yaml
/etc/kubernetes/manifests/kube-apiserver.yaml

#also need to check kubelet manifests and the pods which are running

controlplane $ kubectl -n kube-system get pod -o wide | grep controlplane #running static-pod
calico-kube-controllers-75bdb5b75d-kh2wj   1/1     Running   2 (19m ago)   34d   192.168.0.2   controlplane   <none>           <none>
canal-rc7zk                                2/2     Running   2 (19m ago)   34d   172.30.1.2    controlplane   <none>           <none>
etcd-controlplane                          1/1     Running   2 (19m ago)   34d   172.30.1.2    controlplane   <none>           <none>
kube-apiserver-controlplane                1/1     Running   2 (19m ago)   34d   172.30.1.2    controlplane   <none>           <none>
kube-controller-manager-controlplane       1/1     Running   2 (19m ago)   34d   172.30.1.2    controlplane   <none>           <none>
kube-proxy-6xsx9                           1/1     Running   2 (19m ago)   34d   172.30.1.2    controlplane   <none>           <none>
kube-scheduler-controlplane                1/1     Running   2 (19m ago)   34d   172.30.1.2    controlplane   <none>           <none>


#to see dns

controlplane $ kubectl -n kube-system get deployments.apps 
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-kube-controllers   1/1     1            1           34d
coredns                   2/2     2            2           34d


#to make necessary files
controlplane $ mkdir -p /opt/course/8/
controlplane $ cd /opt/course/8/
controlplane $ vi controlplane-componenet.txt

#answer
kubelet: process
kube-apiserver: static-pod
kube-scheduler: static-pod
kube-controller-manager: static-pod
etcd: static-pod
dns: pod coredns
```

### 85. Create a new namespae named appychip. Create a new network policy named my-policy in the appychip namespace. (1) Network policy should allow PODS within the appychip to connect to each other only on port 80. No other ports should be allowed. (2) No PODs from outside of the appychip should be able to connect to any pods inside the appychip. (11%)

```bash
controlplane $ k create namespace appychip
namespace/appychip created
controlplane $ k get ns
NAME                 STATUS   AGE
appychip             Active   14s

controlplane $ k run nginx --image nginx -n appychip 
pod/nginx created

controlplane $ k get pods -n appychip 
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          14s
```

ns.yaml
```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-policy
  namespace: appychip
spec:
  podSelector: {} #to all the pod in the namespace
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {} #from all the pod in the namespace
    ports:
    - protocol: TCP
      port: 80
```

### 86. Create a pod output-pod which write "congratulations! you have passed CKA Exam" into a file "output-pod.txt". The Pod output-pod should be deleted automatically after writing the text to the file.


```bash
#--rm to remove pod --restart=Never to never allow restart
controlplane $ k run output-pod --image=busybox -it --rm --restart=Never -- /bin/sh -c "echo Congratulations! you have passed the CKA exam" > output-pod.txt
controlplane $ cat output-pod.txt 
Congratulations! you have passed the CKA exam
pod "output-pod" deleted

```


### 87. Deploy a pod named nginx-pod using the nginx:alpine image.


```bash
controlplane ~ ➜  k run nginx-pod --image nginx:alpine
pod/nginx-pod created

controlplane ~ ➜  k get pods
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          10s

controlplane ~ ➜  k describe pods  nginx-pod | grep Image
    Image:          nginx:alpine
    Image ID:       docker.io/library/nginx@sha256:1e67a3c8607fe555f47dc8a72f25424b10273639136c061c508628da3112f90e

```

### 88. Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.

```bash
controlplane ~ ➜  k run --help | grep labels
  # Start a hazelcast pod and set labels "app=hazelcast" and "env=prod" in the container
  kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod"
    -l, --labels='':
        Comma separated labels to apply to the pod. Will override previous values.
```

### 89. Create a namespace named apx-x9984574.

```bash

controlplane ~ ➜  k create namespace apx-x9984574
namespace/apx-x9984574 created
```

### 90. Get the list of nodes in JSON format and store it in a file at /opt/outputs/nodes-z3444kd9.json.


```bash

 k get node -o json > /opt/outputs/nodes-z3444kd9.json
```

### 91. Create a service messaging-service to expose the messaging application within the cluster on port 6379. Use imperative commands.

```bash
controlplane ~ ➜  k expose pod messaging --port 6379 --name messaging-service
service/messaging-service exposed

controlplane ~ ➜  k get svc
NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP    59m
messaging-service   ClusterIP   10.109.0.114   <none>        6379/TCP   5s
```

### 92. Create a deployment named hr-web-app using the image kodekloud/webapp-color with 2 replicas.

```bash
controlplane ~ ➜  k create deployment hr-web-app --image kodekloud/webapp-color --replicas 2
deployment.apps/hr-web-app created

controlplane ~ ➜  k get deployments.apps -w
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hr-web-app   2/2     2            2           11s
```

### 93. Create a static pod named static-busybox on the controlplane node that uses the busybox image and the command sleep 1000.



```bash

k run static-busybox --image busybox --dry-run=client -o yaml --command -- sleep 1000 > static-busybox.yaml #after --command everything is command


controlplane ~ ➜  cat /etc/kubernetes/manifests/static-busybox.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - command:
    - sleep
    - "1000"
    image: busybox
    name: static-busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

controlplane ~ ➜  k get pods
NAME                          READY   STATUS    RESTARTS   AGE
hr-web-app-5d6b77db78-5ttkd   1/1     Running   0          11m
hr-web-app-5d6b77db78-9mvcl   1/1     Running   0          11m
messaging                     1/1     Running   0          24m
nginx-pod                     1/1     Running   0          29m
static-busybox-controlplane   1/1     Running   0          14s
```

### 94.Create a POD in the finance namespace named temp-bus with the image redis:alpine.

```bash
controlplane ~ ➜  k run temp-bus --image redis:alpine -n finance
pod/temp-bus created

controlplane ~ ➜  k get pods -n finance 
NAME       READY   STATUS    RESTARTS   AGE
temp-bus   1/1     Running   0          10s
```

### 95. A new application orange is deployed. There is something wrong with it. Identify and fix the issue.

```bash
controlplane ~ ➜  k logs orange init-myservice #to check the logs of specific pods.
sh: sleeeep: not found

controlplane ~ ➜  k edit pods orange #edit the files remove sleep
error: pods "orange" is invalid
A copy of your changes has been stored to "/tmp/kubectl-edit-3372500299.yaml"
error: Edit cancelled, no valid changes were saved.


controlplane ~ ➜  kubectl replace --force -f /tmp/kubectl-edit-3372500299.yaml
pod "orange" deleted
pod/orange replaced

controlplane ~ ➜  k get pods -w
NAME                          READY   STATUS    RESTARTS   AGE
hr-web-app-5d6b77db78-5ttkd   1/1     Running   0          21m
hr-web-app-5d6b77db78-9mvcl   1/1     Running   0          21m
messaging                     1/1     Running   0          34m
nginx-pod                     1/1     Running   0          39m
orange                        1/1     Running   0          7s
static-busybox-controlplane   1/1     Running   0          10m

```

### 96. Expose the hr-web-app as service hr-web-app-service application on port 30082 on the nodes on the cluster. The web application listens on port 8080.

```bash
controlplane ~ ✖ k expose deployment hr-web-app --name hr-web-app-service --type NodePort --port 8080
service/hr-web-app-service exposed

controlplane ~ ➜  k get svc
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hr-web-app-service   NodePort    10.108.151.157   <none>        8080:31644/TCP   6s
kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP          87m
messaging-service    ClusterIP   10.109.0.114     <none>        6379/TCP         28m

# we can't specify the nodeport directly so we need to edit in the yaml file

controlplane ~ ➜  k edit svc hr-web-app-service 
service/hr-web-app-service edited

controlplane ~ ➜  k get svc
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hr-web-app-service   NodePort    10.108.151.157   <none>        8080:30082/TCP   115s
kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP          89m
messaging-service    ClusterIP   10.109.0.114     <none>        6379/TCP         29m
```

### 97. Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt. The osImages are under the nodeInfo section under status of each node.

```bash
controlplane ~ ➜  kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}
'
Ubuntu 22.04.4 LTS
controlplane ~ ➜  kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt

controlplane ~ ➜  cat /opt/outputs/nodes_os_x43kj56.txt
Ubuntu 22.04.4 LTS
```

### 98. Create a Persistent Volume with the given specification: -

##### Volume name: pv-analytics

##### Storage: 100Mi

##### Access mode: ReadWriteMany

##### Host path: /pv/data-analytics



```bash
controlplane ~ ➜  cat pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/data-analytics

controlplane ~ ✖ k apply -f pv.yaml
persistentvolume/pv-analytics created

controlplane ~ ➜  k get pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-analytics   100Mi      RWX            Retain           Available                          <unset>                          5s



controlplane ~ ➜  k describe pv pv-analytics 
Name:            pv-analytics
Labels:          <none>
Annotations:     <none>
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    
Status:          Available
Claim:           
Reclaim Policy:  Retain
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        100Mi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /pv/data-analytics
    HostPathType:  
Events:            <none>

```

### 99. Take a backup of the etcd cluster and save it to /opt/etcd-backup.db.


```bash
controlplane ~ ➜  cat /etc/kubernetes/manifests/etcd.yaml | grep file
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    seccompProfile:

controlplane ~ ✖ ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key   snapshot save /opt/etcd-backup.db
Snapshot saved at /opt/etcd-backup.db

controlplane ~ ➜  export ETCDCTL_API=3
etcdctl --write-out=table snapshot status  /opt/etcd-backup.db
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 46f0055a |     2808 |       1005 |     2.3 MB |
+----------+----------+------------+------------+

```




### 100. Create a Pod called redis-storage with image: redis:alpine with a Volume of type emptyDir that lasts for the life of the Pod.

Pod named 'redis-storage' created

Pod 'redis-storage' uses Volume type of emptyDir

Pod 'redis-storage' uses volumeMount with mountPath = /data/redis

https://kubernetes.io/docs/concepts/storage/volumes/


```bash
controlplane ~ ➜  k run redis-storage --image redis:alpine --dry-run=client -o yaml > redis-storage.yaml

controlplane ~ ➜  vi redis-storage.yaml 

controlplane ~ ➜  cat redis-storage.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis-storage
  name: redis-storage
spec:
  containers:
  - image: redis:alpine
    name: redis-storage
    volumeMounts:
    - mountPath: /data/redis
      name: cache-volume
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
  - name: cache-volume
    emptyDir: {} #since their is no size limits

status: {}

controlplane ~ ➜  k apply -f redis-storage.yaml 
pod/redis-storage created

controlplane ~ ✖ k get pods -w
NAME            READY   STATUS    RESTARTS   AGE
redis-storage   1/1     Running   0          10s

```

### 101. Create a new pod called super-user-pod with image busybox:1.28. Allow the pod to be able to set system_time.

it's about security context. https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

```bash
controlplane ~ ➜  k run super-user-pod --image busybox:1.28 --dry-run=client -o yaml> super-pod.yaml

controlplane ~ ➜  vi super-pod.yaml 

controlplane ~ ➜  cat super-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - image: busybox:1.28
    name: super-user-pod
    command: ["sleep", "4800"]
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```


### 102. A pod definition file is created at /root/CKA/use-pv.yaml. Make use of this manifest file and mount the persistent volume called pv-1. Ensure the pod is running and the PV is bound.


##### mountPath: /data

##### persistentVolumeClaim Name: my-pvc

```bash

controlplane ~ ➜  k get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-1   10Mi       RWO            Retain           Available                          <unset>                          7m26s

controlplane ~ ➜  vi pvc.yaml

controlplane ~ ➜  cat pvc.yaml 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:  #it's important
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi


controlplane ~ ➜  k apply -f pvc.yaml 
persistentvolumeclaim/my-pvc created

controlplane ~ ➜  k get pvc
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS VOLUMEATTRIBUTESCLASS   AGE
my-pvc   Bound    pv-1     10Mi       RWO                           <unset>                 3s


controlplane ~ ➜  cat CKA/use-pv.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: use-pv
  name: use-pv
spec:
  containers:
  - image: nginx
    name: use-pv
    volumeMounts:
    - mountPath: "/data"
      name: mypd
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: my-pvc
status: {}

```

### 103. Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.

```bash
controlplane ~ ➜  k create deployment nginx-deploy --image nginx:1.16 --replicas 1
deployment.apps/nginx-deploy created


controlplane ~ ➜  k create deployment nginx-deploy --image nginx:1.16 --replicas 1
deployment.apps/nginx-deploy created


controlplane ~ ➜  k describe deployments.apps nginx-deploy 
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Fri, 06 Sep 2024 10:47:07 +0000
Labels:                 app=nginx-deploy
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-deploy
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-deploy
  Containers:
   nginx:
    Image:         nginx:1.16
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deploy-858fb84d4b (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  44s   deployment-controller  Scaled up replica set nginx-deploy-858fb84d4b to 1

controlplane ~ ➜  k set image deployment/nginx-deploy nginx=nginx:1.17
deployment.apps/nginx-deploy image updated

controlplane ~ ➜  k describe deployments.apps nginx-deploy 
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Fri, 06 Sep 2024 10:47:07 +0000
Labels:                 app=nginx-deploy
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=nginx-deploy
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-deploy
  Containers:
   nginx:
    Image:         nginx:1.17
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  nginx-deploy-858fb84d4b (0/0 replicas created)
NewReplicaSet:   nginx-deploy-58f87d49 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  2m7s  deployment-controller  Scaled up replica set nginx-deploy-858fb84d4b to 1
  Normal  ScalingReplicaSet  10s   deployment-controller  Scaled up replica set nginx-deploy-58f87d49 to 1
  Normal  ScalingReplicaSet  6s    deployment-controller  Scaled down replica set nginx-deploy-858fb84d4b to 0 from 1

```

### 104. Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.


##### Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

##### Please refer the documentation to see an example. The documentation tab is available at the top right of terminal.


```bash


k auth can-i get pods --namespace=development --as john
k auth can-i create pods --namespace=development --as john
k auth can-i watch pods --namespace=development --as john


```

### 105. Create a nginx pod called nginx-resolver using image nginx, expose it internally with a service called nginx-resolver-service. Test that you are able to look up the service and pod names from within the cluster. Use the image: busybox:1.28 for dns lookup. Record results in /root/CKA/nginx.svc and /root/CKA/nginx.pod.

```bash

k run nginx-resolver --image=nginx #to create an nginx pod

#expose it as a service

k expose pod nginx-resolver --name=nginx-resolver-service --port=80

#verify the service
k describe svc nginx-resolver-service

#to run the service
k run busybox --image=busybox:1.28 -- sleep 4000 #sleep is to run it on the background

#for dns lookup from the pod
k exec busybox -- nslookup nginx-resolver-service > /root/CKA/nginx.svc

#first we need to find out the ip address
k get pods -o wide

k exec busybox -- nslookup <ip-addr>.default.pod.cluster.local > /root/CKA/nginx.pod
#ip address needed to be separated by - instead of . 10-50-192-4

```
search for pod dns in documentation to solve further. 
https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/



### 106. Create a static pod on node01 called nginx-critical with image nginx and make sure that it is recreated/restarted automatically in case of a failure.


##### Use /etc/kubernetes/manifests as the Static Pod path for example.


```bash

#ssh into node

ssh node01

kubectl run nginx-critical --image=nginx --restart=Always --dry-run=client -o yaml

#copy it and paste it like 

cat > /etc/kubernetes/manifests/nginx-critical.yaml #and paste it then ctrl+c


k get pods #to check pods

```

### 107. Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.
Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace.

```bash

k create serviceaccount pvviewer

k create clusterrole pvviewer-role --verb=list --resource=persistentvolumes

k create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer  # default is the namespae

k describe clusterrolebinding pvviewer-role-binding

k run pvviewer --image=redis --dry-run -o yaml > pvviewer.yaml

#then add service account inside the yaml file
```

search for service account in k8s documentation
https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

### 108. List the InternalIP of all nodes of the cluster. Save the result to a file /root/CKA/node_ips.Answer should be in the format: InternalIP of controlplane<space>InternalIP of node01 (in a single line)


```bash
k get nodes -o wide -o json | jp | grep -i internalip -B 100

k get nodes -o json | jp -c 'paths' | grep type | grep -v condition

k get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}' | jq

k get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}' | jq > /root/CKA/node_ips

```

### 109. Create a pod called multi-pod with two containers.
##### Container 1: name: alpha, image: nginx
##### Container 2: name: beta, image: busybox, command: sleep 4800

##### Environment Variables:
###### container 1:
###### name: alpha

###### Container 2:
###### name: beta

```bash

k run multi-pod --image=nginx --dry-run -o yaml > multipod.yaml

```




### 109. Create a Pod called non-root-pod , image: redis:alpine

##### runAsUser: 1000

##### fsGroup: 2000

security context
k get pods <name> -o yaml



### 109. We have deployed a new pod called np-test-1 and a service called np-test-service. Incoming connections to this service are not working. Troubleshoot and fix it.
Create NetworkPolicy, by the name ingress-to-nptest that allows incoming connections to the service over port 80.


Important: Don't delete any current objects deployed.

Important: Don't Alter Existing Objects!

NetworkPolicy: Applied to All sources (Incoming traffic from all pods)?

NetWorkPolicy: Correct Port?

NetWorkPolicy: Applied to correct Pod?


```bash

k run curl --image=alpine/curl --rm -it -- sh #for testing curl
k get pods --show-labels #for getting labels and working in that favour
```




### 110. Taint the worker node node01 to be Unschedulable. Once done, create a pod called dev-redis, image redis:alpine, to ensure workloads are not scheduled to this worker node. Finally, create a new pod called prod-redis and image: redis:alpine with toleration to be scheduled on node01.


key: env_type, value: production, operator: Equal and effect: NoSchedule

```bash
k taint nodes node01 env_type=production:NoSchedule

k run dev-redis --image=redis:alpine

k run prod-redis --image=reedis:alpine --dry-run=client -o yaml > prod-redis.yaml
```

```bash
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```

### 111. Create a pod called hr-pod in hr namespace belonging to the production environment and frontend tier .
image: redis:alpine


Use appropriate labels and create all the required objects if it does not exist in the system already.

labels command


### 112. A kubeconfig file called super.kubeconfig has been created under /root/CKA. There is something wrong with the configuration. Troubleshoot and fix it.

```bash

kubectl get nodes --kubeconfig /root/CKA/super.kubeconfig

cat .kube/config #to see the current kubeconfig file
#or
kubectl config view
```

### 113. We have created a new deployment called nginx-deploy. scale the deployment to 3 replicas. Has the replica's increased? Troubleshoot the issue and fix it.

```bash

kubectl get pods -n kube-system #to check the controller manager and pods.

```

### 114. Create a new deployment called nginx-deploy, with image nginx:1.16 and 8 replica. There are 5 worker node in cluster. Please make sure no pod will get deployed on 2 worker node, mentioned below: 
##### Worker-node-1
##### Worker-node-2

Cordon both of the nodes and do deployment and uncordon the nodes


### 115. There are 3 Node in the cluster, create DaemonSet (Name: my-pod, Image: nginx) on each node except one (Worker-node-3).

Taint the specific node
```bash
kubectl taint node worker-node-3 env=qa:NoSchedule

kubectl describe node worker-node-3-cgaga | grep -i taint
```

### 116. Generate a file (CKA007.txt) with details about the available size of all the node in the k8s cluster using a custom column, format as mentioned below.

name  | Available_memory | available_cpu
node1 | ...              | ...
node2 | ...              | ...


```bash

kubectl get nodes -o json

kubectl get node -o custom-columns=NAME:.metadata.name.AVAILABLE_MEMORY:.status.allocatable.memory,AVAILABLE_CPU:.status.allocatable.cpu > CKA007.txt
```

## Troubleshooting in k8s

Check Kubernetes Components, Insepct Logs, Rersource Usage, Networking, Health Checks, Cluster configuration, Check Events, Version Compatibility, Community Resources, Documentation and Best practices


### 117. Troubleshooting Test 1: A simple 2 tier application is deployed in the alpha namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

```bash

controlplane ~ ➜  k get pods -n alpha 
NAME                           READY   STATUS    RESTARTS   AGE
webapp-mysql-9b567db7f-h4gkp   1/1     Running   0          2m43s
mysql                          1/1     Running   0          2m43s

controlplane ~ ➜  kubectl config set-context --current --namespace alpha
Context "default" modified.

controlplane ~ ➜  k get pods
NAME                           READY   STATUS    RESTARTS   AGE
webapp-mysql-9b567db7f-h4gkp   1/1     Running   0          3m50s
mysql                          1/1     Running   0          3m50s


controlplane ~ ➜  k get deployments.apps 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
webapp-mysql   1/1     1            1           4m27s

controlplane ~ ➜  k get pods 
NAME                           READY   STATUS    RESTARTS   AGE
webapp-mysql-9b567db7f-h4gkp   1/1     Running   0          4m57s
mysql                          1/1     Running   0          4m57s

controlplane ~ ➜  k get svc
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
mysql         ClusterIP   10.43.85.40    <none>        3306/TCP         5m6s
web-service   NodePort    10.43.110.18   <none>        8080:30081/TCP   5m6s


controlplane ~ ➜  curl http://localhost:30081
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #ff3f3f;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">

  
    <img src="/static/img/failed.png">
    <!-- <h1> DATABASE CONNECTION FAILED !!</h1> -->
  

  
    <h2> Environment Variables: DB_Host=mysql-service; DB_Database=Not Set; DB_User=root; DB_Password=paswrd; 2003: Can&#39;t connect to MySQL server on &#39;mysql-service:3306&#39; (-2 Name does not resolve) </h2>
  

  
    <p> From webapp-mysql-9b567db7f-h4gkp!</p>
  

  

</div>


controlplane ~ ➜  k describe deployments.apps webapp-mysql 
Name:                   webapp-mysql
Namespace:              alpha
CreationTimestamp:      Sat, 07 Sep 2024 17:52:51 +0000
Labels:                 name=webapp-mysql
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp-mysql
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp-mysql
  Containers:
   webapp-mysql:
    Image:      mmumshad/simple-webapp-mysql
    Port:       8080/TCP
    Host Port:  0/TCP
    Environment:
      DB_Host:      mysql-service  #the service name is mysql not sql
      DB_User:      root
      DB_Password:  paswrd
    Mounts:         <none>
  Volumes:          <none>
  Node-Selectors:   <none>
  Tolerations:      <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   webapp-mysql-9b567db7f (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  6m12s  deployment-controller  Scaled up replica set webapp-mysql-9b567db7f to 1

```


### 118. Troubleshooting Test 2: The same 2 tier application is deployed in the beta namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.


```bash

controlplane ~ ➜  kubectl config set-context --current --namespace beta
Context "default" modified.

controlplane ~ ➜  k get pods
NAME                           READY   STATUS    RESTARTS   AGE
webapp-mysql-9b567db7f-fqszt   1/1     Running   0          76s
mysql                          1/1     Running   0          76s

controlplane ~ ➜  k get deployments.apps 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
webapp-mysql   1/1     1            1           81s

controlplane ~ ➜  k get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
mysql-service   ClusterIP   10.43.132.87   <none>        3306/TCP         2m13s
web-service     NodePort    10.43.25.92    <none>        8080:30081/TCP   2m13s

controlplane ~ ➜  k describe deployments.apps webapp-mysql 
Name:                   webapp-mysql
Namespace:              beta
CreationTimestamp:      Sat, 07 Sep 2024 18:04:16 +0000
Labels:                 name=webapp-mysql
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp-mysql
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp-mysql
  Containers:
   webapp-mysql:
    Image:      mmumshad/simple-webapp-mysql
    Port:       8080/TCP
    Host Port:  0/TCP
    Environment:
      DB_Host:      mysql-service
      DB_User:      root
      DB_Password:  paswrd
    Mounts:         <none>
  Volumes:          <none>
  Node-Selectors:   <none>
  Tolerations:      <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   webapp-mysql-9b567db7f (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  3m21s  deployment-controller  Scaled up replica set webapp-mysql-9b567db7f to 1

controlplane ~ ➜  k describe svc mysql-service 
Name:              mysql-service
Namespace:         beta
Labels:            <none>
Annotations:       <none>
Selector:          name=mysql
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.132.87
IPs:               10.43.132.87
Port:              <unset>  3306/TCP
TargetPort:        8080/TCP
Endpoints:         10.42.0.11:8080 #mistake in the  port. port is 3306
Session Affinity:  None
Events:            <none>

controlplane ~ ➜  k get pods --show-labels 
NAME                           READY   STATUS    RESTARTS   AGE     LABELS
webapp-mysql-9b567db7f-fqszt   1/1     Running   0          4m17s   name=webapp-mysql,pod-template-hash=9b567db7f
mysql                          1/1     Running   0          4m17s   name=mysql


controlplane ~ ➜  k get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
webapp-mysql-9b567db7f-fqszt   1/1     Running   0          5m22s   10.42.0.12   controlplane   <none>           <none>
mysql                          1/1     Running   0          5m22s   10.42.0.11   controlplane   <none>           <none>


controlplane ~ ➜  k edit svc mysql-service 
service/mysql-service edited

controlplane ~ ➜  k get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
web-service     NodePort    10.43.25.92    <none>        8080:30081/TCP   8m6s
mysql-service   ClusterIP   10.43.132.87   <none>        3306/TCP         8m6s

controlplane ~ ➜  k describe svc mysql-service 
Name:              mysql-service
Namespace:         beta
Labels:            <none>
Annotations:       <none>
Selector:          name=mysql
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.132.87
IPs:               10.43.132.87
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.42.0.11:3306
Session Affinity:  None
Events:            <none>
```


### 119. Troubleshooting Test 3: The same 2 tier application is deployed in the gamma namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed or unresponsive. Troubleshoot and fix the issue.


Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.


```bash
controlplane ~ ➜  k get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
mysql-service   ClusterIP   10.43.255.33   <none>        3306/TCP         54s
web-service     NodePort    10.43.183.46   <none>        8080:30081/TCP   54s

controlplane ~ ➜  k describe svc mysql-service 
Name:              mysql-service
Namespace:         gamma
Labels:            <none>
Annotations:       <none>
Selector:          name=sql00001 #issue with pod selector
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.255.33
IPs:               10.43.255.33
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>

controlplane ~ ➜  k describe svc web-service 
Name:                     web-service
Namespace:                gamma
Labels:                   <none>
Annotations:              <none>
Selector:                 name=webapp-mysql
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.183.46
IPs:                      10.43.183.46
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30081/TCP
Endpoints:                10.42.0.14:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

controlplane ~ ➜  k get pods --show-labels 
NAME                           READY   STATUS    RESTARTS   AGE     LABELS
webapp-mysql-9b567db7f-982bz   1/1     Running   0          5m23s   name=webapp-mysql,pod-template-hash=9b567db7f
mysql                          1/1     Running   0          5m23s   name=mysql

```



```bash
controlplane ~ ➜  k describe deployments.apps webapp-mysql 
Name:                   webapp-mysql
Namespace:              delta
CreationTimestamp:      Sat, 07 Sep 2024 18:21:13 +0000
Labels:                 name=webapp-mysql
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp-mysql
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp-mysql
  Containers:
   webapp-mysql:
    Image:      mmumshad/simple-webapp-mysql
    Port:       8080/TCP
    Host Port:  0/TCP
    Environment:
      DB_Host:      mysql-service
      DB_User:      sql-user  #error in the user
      DB_Password:  paswrd
    Mounts:         <none>
  Volumes:          <none>
  Node-Selectors:   <none>
  Tolerations:      <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   webapp-mysql-57dc5df9f9 (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  9m29s  deployment-controller  Scaled up replica set webapp-mysql-57dc5df9f9 to 1
```

### 120. Troubleshooting Test 5: The same 2 tier application is deployed in the epsilon namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

```bash
controlplane ~ ➜  k describe pods mysql 
Name:             mysql
Namespace:        epsilon
Priority:         0
Service Account:  default
Node:             controlplane/192.25.200.9
Start Time:       Sat, 07 Sep 2024 18:32:20 +0000
Labels:           name=mysql
Annotations:      <none>
Status:           Running
IP:               10.42.0.18
IPs:
  IP:  10.42.0.18
Containers:
  mysql:
    Container ID:   containerd://d2d769c3a115f0193e143e64ba8106542cc4aeaa0ebf5b2d65db52047624561a
    Image:          mysql:5.6
    Image ID:       docker.io/library/mysql@sha256:20575ecebe6216036d25dab5903808211f1e9ba63dc7825ac20cb975e34cfcae
    Port:           3306/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 07 Sep 2024 18:32:22 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      MYSQL_ROOT_PASSWORD:  passwooooorrddd  #error here
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-c98dx (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-c98dx:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m10s  default-scheduler  Successfully assigned epsilon/mysql to controlplane
  Normal  Pulled     3m10s  kubelet            Container image "mysql:5.6" already present on machine
  Normal  Created    3m10s  kubelet            Created container mysql
  Normal  Started    3m9s   kubelet            Started container mysql
```


### 121. Troubleshooting Test 6: The same 2 tier application is deployed in the zeta namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.


Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.



### troubleshooting control plane

https://kubernetes.io/docs/tasks/debug/debug-cluster/

check 

```bash

kubectl get nodes #to see the nodes are healthy

kubectl get pods #to see the nodes

kubectl get pods -n kube-system #to see the pods in kube-system namespace is healthy

service kube-apiserver status #to check service status

service kube-controller-manager status

service kubelet status #in worker nodes

service kube-proxy status #in worker nodes

kubectl logs kube-apiserver-master -n kube-system


kubectl logs kube-scheduler-controlplane -n kube-system
```


### 124. The cluster is broken. We tried deploying an application but it's not working. Troubleshoot and fix the issue.

Checked pods, deployement and rc


```bash
controlplane ~ ➜  k describe pods app-5f58858856-dnjfz 
Name:             app-5f58858856-dnjfz
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>  #node is assigned to none issue is their it's schedulers job
Labels:           app=app
                  pod-template-hash=5f58858856
Annotations:      <none>
Status:           Pending
IP:               
IPs:              <none>
Controlled By:    ReplicaSet/app-5f58858856
Containers:
  nginx:
    Image:        nginx:alpine
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8s52m (ro)
Volumes:
  kube-api-access-8s52m:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>



controlplane ~ ➜  k get pods -n kube-system 
NAME                                   READY   STATUS             RESTARTS      AGE
coredns-768b85b76f-4mp4n               1/1     Running            0             11m
coredns-768b85b76f-96hwn               1/1     Running            0             11m
etcd-controlplane                      1/1     Running            0             11m
kube-apiserver-controlplane            1/1     Running            0             11m
kube-controller-manager-controlplane   1/1     Running            0             11m
kube-proxy-47nhv                       1/1     Running            0             11m
kube-scheduler-controlplane            0/1     CrashLoopBackOff   4 (87s ago)   3m7s
#crashloop backoff


controlplane ~ ➜  k describe pods -n kube-system kube-scheduler-controlplane 
Name:                 kube-scheduler-controlplane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 controlplane/192.28.249.3
Start Time:           Sat, 07 Sep 2024 20:01:53 +0000
Labels:               component=kube-scheduler
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: e4007249bf4f8d68970f15caac4f8d27
                      kubernetes.io/config.mirror: e4007249bf4f8d68970f15caac4f8d27
                      kubernetes.io/config.seen: 2024-09-07T20:10:00.399309282Z
                      kubernetes.io/config.source: file
Status:               Running
SeccompProfile:       RuntimeDefault
IP:                   192.28.249.3
IPs:
  IP:           192.28.249.3
Controlled By:  Node/controlplane
Containers:
  kube-scheduler:
    Container ID:  containerd://1aece1f54a4fffff76520aae1092a0d7203b7b6e3d2053c20a3b4e8480843fc6
    Image:         registry.k8s.io/kube-scheduler:v1.30.0
    Image ID:      registry.k8s.io/kube-scheduler@sha256:2353c3a1803229970fcb571cffc9b2f120372350e01c7381b4b650c4a02b9d67
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-schedulerrrr
      --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
      --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
      --bind-address=127.0.0.1
      --kubeconfig=/etc/kubernetes/scheduler.conf
      --leader-elect=true
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       StartError
      Message:      failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
      Exit Code:    128
      Started:      Thu, 01 Jan 1970 00:00:00 +0000
      Finished:     Sat, 07 Sep 2024 20:13:30 +0000
    Ready:          False
    Restart Count:  5
    Requests:
      cpu:        100m
    Liveness:     http-get https://127.0.0.1:10259/healthz delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get https://127.0.0.1:10259/healthz delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/scheduler.conf from kubeconfig (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kubeconfig:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/scheduler.conf
    HostPathType:  FileOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:
  Type     Reason   Age                     From     Message
  ----     ------   ----                    ----     -------
  Normal   Created  3m22s (x4 over 4m18s)   kubelet  Created container kube-scheduler
  Warning  Failed   3m22s (x4 over 4m18s)   kubelet  Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
  Warning  BackOff  2m49s (x12 over 4m16s)  kubelet  Back-off restarting failed container kube-scheduler in pod kube-scheduler-controlplane_kube-system(e4007249bf4f8d68970f15caac4f8d27)
  Normal   Pulled   2m38s (x5 over 4m18s)   kubelet  Container image "registry.k8s.io/kube-scheduler:v1.30.0" already present on machine


### "kube-schedulerrrr" is right


controlplane ~ ➜  cat /etc/kubernetes/manifests/kube-scheduler.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-schedulerrrr  # here is the error
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    image: registry.k8s.io/kube-scheduler:v1.30.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10259
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}


```

### 125.Even though the deployment was scaled to 2, the number of PODs does not seem to increase. Investigate and fix the issue.


Inspect the component responsible for managing deployments and replicasets. 



```bash
controlplane ~ ➜  k get pods -n kube-system 
NAME                                   READY   STATUS             RESTARTS      AGE
coredns-768b85b76f-4mp4n               1/1     Running            0             23m
coredns-768b85b76f-96hwn               1/1     Running            0             23m
etcd-controlplane                      1/1     Running            0             23m
kube-apiserver-controlplane            1/1     Running            0             23m
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   6 (94s ago)   7m30s
kube-proxy-47nhv                       1/1     Running            0             23m
kube-scheduler-controlplane            1/1     Running            0             8m11s

controlplane ~ ➜  k describe pods -n kube-system kube-controller-manager-controlplane






```




### Worker Node Failure

```bash

kubectl get nodes

kubectl describe node worker-1 #status of worker node

top #to check memory

service kubelet status #status of kubelet

service kubelet start #to start the kubelet

sudo journalctl -u kubelet #kubelet logs

#check kubelet certificates

openssl x509 -in /var/lib/kubelet/worker-1.crt -text


  cat /etc/kubernetes/kubelet.conf #location of kubelet conf file


```

```bash

node01 ~ ➜  cat /var/lib/kubelet/config.yaml 
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/WRONG-CA-FILE.crt  #this should be the file
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
containerRuntimeEndpoint: ""
cpuManagerReconcilePeriod: 0s
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMaximumGCAge: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
    text:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
resolvConf: /run/systemd/resolve/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s


node01 ~ ➜  ls /etc/kubernetes/pki/
ca.crt
#it's shoudl be ca.crt


service kubelet restart
```

### 126. List all persistent volumes sorted by capacity, saving the full kubectl output to /opt/pv/pv_list.txt

```bash
kubectl get pv --sort-by=.spec.capacity.storage > /opt/pv/pv_list.txt
```


### 127. Retrieve the logs from the pod named 'webpod' search for any occurrences of the word 'failed' within those logs, and then save those findings into a file located at '/opt/errorlogs.txt'

```bash
k logs webpod | grep -i failed > /opt/errorlogs.txt
```

### 128. Create a nginx pod named nginxpod with label env=prod in production namespace

use --labels option from k run

### 129. Create a pod named 'non-root-pod' using the redis image, with the 'runAsUser' set to 1000 and 'fsGroup' set to 2000.

use security context

### 130. create a mult-container pod named frontend with the image of nginx+redis


### 131. Identify pods with the label app=mysql that are executing high CPU workloads and write the name of the pod consuming most CPU to the file /opt/toppods.yaml

```bash

k top pods #high consumption

k top pods -l app=mysql --sort-by=cpu

```

### 132. Verify the count of nodes that are ready (excluding those tainted with NoSchedule) and write the number to /opt/nodecount.txt


```bash
k describe nodes | grep -A 5 taints #after 5 lines
```

### 133. Schedule a pod as follows. Name: frontend, Image: nginx, Node selector: disk=ssd

```bash
k get nodes --show-labels | grep disk

```

add node selector inside the spec

```bash
nodeSelector:
  disk: ssh
```

from cka kille code
### 134. Create a new role binding named sa-creator-binding that will bind to the sa-creator role and apply to a user named Sandra in the default namespace.

Verify that you can create service accounts with the Sandra user using auth can-i

```bash
controlplane $ k create rolebinding sa-creator-binding --role=sa-creator --user=Sandra
rolebinding.rbac.authorization.k8s.io/sa-creator-binding created
controlplane $ k get role,rolebinding
NAME                                        CREATED AT
role.rbac.authorization.k8s.io/sa-creator   2024-09-09T12:24:30Z

NAME                                                       ROLE              AGE
rolebinding.rbac.authorization.k8s.io/sa-creator-binding   Role/sa-creator   14s
controlplane $ k auth can-i create sa --namespace default --as Sandra
yes
controlplane $ k auth can-i create po --namespace default --as Sandra
no
```

### 135. Create a service account named dev Create a role binding that will bind the view cluster role to the newly created dev service account in the default namespace. Verify that the dev service account can view pods and services in the default namespace with auth can-i

```bash
controlplane $ k create sa dev
serviceaccount/dev created
controlplane $ k get sa
NAME      SECRETS   AGE
default   0         38d
dev       0         5s

controlplane $ kubectl create rolebinding admin-binding --role=view --serviceaccount=default:dev
rolebinding.rbac.authorization.k8s.io/admin-binding created

controlplane $ kubectl auth can-i get svc --namespace default --as=system:serviceaccount:default:dev
yes
controlplane $ 
```

### 136. Create a new cluster role named “acme-corp-clusterrole” that can create deployments, replicasets and daemonsets.

```bash
controlplane $ k create role acme-corp-clusterrole --verb=create --resource=deploy,rc,ds
role.rbac.authorization.k8s.io/acme-corp-clusterrole created
controlplane $ k get role
NAME                    CREATED AT
acme-corp-clusterrole   2024-09-09T12:45:52Z
```

### 137. Bind the cluster role 'acme-corp-clusterrole' to the service account ’secure-sa’ making sure the 'secure-sa' service account can only create the assigned resources within the default namespace and nowhere else.

Verify that the service account can only create deployments, replicaSets, and daemonSets in the default namespace.

```bash
controlplane $ k create rolebinding acme-corp-clusterrole-binding --role acme-corp-clusterrole --serviceaccount=default:secure-s
a --namespace default
rolebinding.rbac.authorization.k8s.io/acme-corp-clusterrole-binding created
controlplane $ k get rolebindings.rbac.authorization.k8s.io -n default 
NAME                            ROLE                         AGE
acme-corp-clusterrole-binding   Role/acme-corp-clusterrole   12s


#namespace should come first


# using 'auth can-i', verify that you can create deployments as the 'secure-sa' service account in the default namespace
kubectl auth can-i create deploy --namespace default --as=system:serviceaccount:default:secure-sa

# using 'auth can-i', verify that you CANNOT delete daemonSets as the 'secure-sa' service account in the kube-system namespace 
kubectl auth can-i delete ds --namespace kube-system --as=system:serviceaccount:default:secure-sa
```

### 138. Run the appropriate command to check the current version of the API server, controller manager, scheduler, kube-proxy, CoreDNS, and etcd.

The output should look similar to the following:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.30.0    v1.30.1
kube-controller-manager   controlplane   v1.30.0    v1.30.1
kube-scheduler            controlplane   v1.30.0    v1.30.1
kube-proxy                               1.30.0     v1.30.1
CoreDNS                                  v1.11.1    v1.11.1
etcd                      controlplane   3.5.12-0   3.5.12-0

kubeadm -h

```bash
kudeadm upgrade plan
```

### 139. Create a new service account named ’secure-sa’ in the default namespace that will not automatically mount the service account token.

```bash
controlplane $ k create sa secure-sa --dry-run=client -o yaml > sa.yaml
controlplane $ vi sa.yaml
controlplane $ cat sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: null
  name: secure-sa
automountServiceAccountToken: false
controlplane $ k apply -f sa.yaml 
serviceaccount/secure-sa created
controlplane $ k get sa
NAME        SECRETS   AGE
default     0         38d
secure-sa   0         6s
```

### 140. Create a pod that uses the ‘secure-sa’ service account. Make sure the token is not exposed to the pod.

```bash
controlplane $ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod
  name: pod
spec:
  serviceAccountName: secure-sa
  containers:
  - image: nginx
    name: pod

controlplane $ k apply -f pod.yaml 
pod/pod created
controlplane $ k get pods -w
NAME   READY   STATUS              RESTARTS   AGE
pod    0/1     ContainerCreating   0          9s
pod    1/1     Running             0          10s

# Verify that the service account token is NOT mounted to the pod

^Ccontrolplane $ kubectl exec secure-pod -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
Error from server (NotFound): pods "secure-pod" not found
controlplane $ 
```

### 141. View the file in your home directory named pod.yaml . Apply the correct toleration to this pod manifest in order for it to successfully get scheduled to node01.

After modifying the pod.yaml file, run the command k apply -f ~/pod.yaml to create the pod.

Then, run the command k get po -o wide to verify that the pod status is Running

HINT: The key, value, and effect have to match the taint

```bash
controlplane $ k apply -f pod.yaml 
pod/nginx created
controlplane $ k get pods -o wide 
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          10s   192.168.1.4   node01   <none>           <none>
controlplane $ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "special-user"
    effect: "NoSchedule"
```


### 142. In the namespace named 012963bd , create a pod named ctrl-pod which uses the nginx image. This pod should be scheduled to the control plane node.

Ensure that the pod is scheduled to the controlplane node.

```bash
controlplane $ k get pods -n 012963bd 
NAME       READY   STATUS              RESTARTS   AGE
ctrl-pod   0/1     ContainerCreating   0          9s
controlplane $ k get pods -n 012963bd 
NAME       READY   STATUS    RESTARTS   AGE
ctrl-pod   1/1     Running   0          13s
controlplane $ cat pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ctrl-pod
  name: ctrl-pod
  namespace: 012963bd
spec:
  nodeName: controlplane  #correct way is using nodeaffinity but this is easier 
  #since their is no required during scheduling rules
  containers:
  - image: nginx
    name: ctrl-pod
    resources: {}
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
controlplane $ 
```


### 145. Create a pod with one container that will log to STDOUT

Use kubectl to view the logs from this container within the pod named "pod-logging"


```bash
cat << EOF > pod-logging.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-logging
spec:
  containers:
  - name: main
    image: busybox
    args: [/bin/sh, -c, 'while true; do echo $(date); sleep 1; done']
EOF

controlplane $ k logs pod-logging
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
controlplane $ 
controlplane $ k logs pod-logging -f
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024
Mon Sep 9 15:42:05 UTC 2024

```

### 146. Create a pod that will have two containers, one main container and another sidecar container that will collect the main containers logs

Using kubectl, view the logs from the container named "sidecar"

```bash

cat << EOF > pod-logging-sidecar.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-logging-sidecar
spec:
  containers:
  - image: busybox
    name: main
    args: [ 'sh', '-c', 'while true; do echo "$(date)\n" >> /var/log/main-container.log; sleep 5; done' ]
    volumeMounts:
      - name: varlog
        mountPath: /var/log
  - name: sidecar
    image: busybox
    args: [ /bin/sh, -c, 'tail -f /var/log/main-container.log' ]
    volumeMounts:
      - name: varlog
        mountPath: /var/log
  volumes:
    - name: varlog
      emptyDir: {}
EOF


controlplane $ k logs pod-logging-sidecar -c sidecar
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
controlplane $ 

controlplane $ k logs pod-logging-sidecar --all-containers 
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
controlplane $ k logs pod-logging-sidecar --all-containers -f
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
Mon Sep  9 15:44:28 UTC 2024\n
^C
```

### 150. Create a deployment named "mysql" that uses the image "mysql:8".

View the deployment and pod, and find out why it's not running

Fix the deployment in order to get the pod in a running state.

```bash
controlplane $ k logs mysql-5479478c76-l2b4k 
2024-09-09 15:49:31+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.2-1.el9 started.
2024-09-09 15:49:31+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2024-09-09 15:49:31+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.2-1.el9 started.
2024-09-09 15:49:31+00:00 [ERROR] [Entrypoint]: Database is uninitialized and password option is not specified
    You need to specify one of the following as an environment variable:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_ALLOW_EMPTY_PASSWORD
    - MYSQL_RANDOM_ROOT_PASSWORD

k create deploy mysql --image mysql:8
# in deployment, add
env:
  - name: MY_SQL_PASSWORD
  value: "password"
```

### 151. Create a configmap named redis-config . Within the configMap, use the key maxmemory with value 2mb and key maxmemory-policy with value allkeys-lru .

HINT: try k create cm -h for command options and examples

```bash

controlplane $ k create cm redis-config --from-literal=maxmemory=2mb --from-literal=maxmemory-policy=allkeys-lru
configmap/redis-config created
controlplane $ k describe configmaps redis-config 
Name:         redis-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
maxmemory:
----
2mb
maxmemory-policy:
----
allkeys-lru

BinaryData
====

Events:  <none>
controlplane $ 
```

### 152. Create a pod named redis-pod that uses the image redis:7 and exposes port 6379 . Use the command redis-server /redis-master/redis.conf to store redis configuration data and store this in an emptyDir volume.

Mount the redis-config configmap data to the pod for use within the container.

HINT: create the pod YAML with a --dry-run using the following command:

```bash

kubectl run redis-pod --image=redis:7 --port=6379 --dry-run=client -o yaml > redis-pod.yaml


apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
spec:
  containers:
  - name: redis
    image: redis:7
    ports:
    - containerPort: 6379
    command: [ "redis-server", "/redis-master/redis.conf" ]
    volumeMounts:
    - name: redis-storage
      mountPath: /redis-master
    - name: redis-config
      mountPath: /redis-master/redis.conf
      subPath: redis.conf
  volumes:
  - name: redis-storage
    emptyDir: {}
  - name: redis-config
    configMap:
      name: redis-config
```

### 153. Create a deployment named “apache” that uses the image httpd . Using only kubectl , change the image from httpd to httpd:2.4.54 . List the events of the replicasets in the cluster. Using kubectl , roll back to a previous version of the deployment (the deployment with the image httpd ).



```bash
controlplane $ # create a deployment named "apache" that uses the image 'httpd'
controlplane $ kubectl create deploy apache --image httpd
deployment.apps/apache created
controlplane $ 
controlplane $ # list the deployment and the pods in that deployment
controlplane $ kubectl get deploy,po
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/apache   0/1     0            0           0s

NAME                         READY   STATUS    RESTARTS   AGE
pod/apache-6c8d9bff6-8829q   0/1     Pending   0          0s
controlplane $ 


controlplane $ kubectl describe rs
Name:           apache-6c8d9bff6
Namespace:      default
Selector:       app=apache,pod-template-hash=6c8d9bff6
Labels:         app=apache
                pod-template-hash=6c8d9bff6
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/apache
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=apache
           pod-template-hash=6c8d9bff6
  Containers:
   httpd:
    Image:         httpd
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  29s   replicaset-controller  Created pod: apache-6c8d9bff6-8829q


Name:           apache-986db45df
Namespace:      default
Selector:       app=apache,pod-template-hash=986db45df
Labels:         app=apache
                pod-template-hash=986db45df
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 2
Controlled By:  Deployment/apache
Replicas:       1 current / 1 desired
Pods Status:    0 Running / 1 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=apache
           pod-template-hash=986db45df
  Containers:
   httpd:
    Image:         httpd:2.4.54
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  1s    replicaset-controller  Created pod: apache-986db45df-gqtw5

```

for rolling out the changes 


```bash
# view the rollout history
kubectl rollout history deploy apache

# roll back to the previous rollout
kubectl rollout undo deploy apache

# view the status of the rollout
kubectl rollout status deploy apache

```


### 154. Change the service cluster IP range that's given to each service to 100.96.0.0/12.

```bash
# change the api server command that hands out service addresses for the cluster
vim /etc/kubernetes/manifests/kube-apiserver.yaml 


# kube-apiserver.yaml
...
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=172.30.1.2
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=100.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
...

```

### 155. Change the IP address associated with the cluster's DNS Service.

It's the kube-dns service in the kube-system namespace. k -n kube-system edit svc kube-dns

```bash
# in the service YAML, modify the 'strategy'. save and quit to apply the changes!
spec:
  clusterIP: 100.96.0.10
  clusterIPs:
  - 100.96.0.10
  internalTrafficPolicy: Cluster
...
```

### 156. Create an Ingress resource that will allow you to resolve the domain name hello.com to the service named apache-svc over port 80 .

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: apache-svc
            port:
              number: 80
```

### 157. Change the restartPolicy to prevent the pod from automatically restarting.

```bash
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - image: busybox
    name: busybox
  restartPolicy: Never # change this from Always to Never
```

### 158. Create a Persistent Volume (PV) named pv-volume that has the following specifications:

a Delete persistentVolumeReclaimPolicy
Uses the strageClass named local-path
References a claim named pv-claim in the default namespace
Uses hostPath volume type, at path /mnt/data
Has a capacity of 1Gi
Access mode is set to ReadWriteOnce
Once you've created the PV, list all the persistentvolumes in the cluster.


```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume
spec:
  persistentVolumeReclaimPolicy: Delete
  storageClassName: "local-path"
  claimRef:
    name: pv-claim
    namespace: default
  hostPath:
    path: "/mnt/data"
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
```

### 159.Create a Persistent Volume Claim (PVC) named pvc-claim that has the following specifications:

Uses the storageClass named local-path
Access mode set to ReadWriteOnce
Requests 1Gi of storage
Once you've created the PVC, list all the persistentvolumeclaims in the cluster.


```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-claim
  namespace: default
spec:
  storageClassName: "local-path"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```


### 160. Now that we have created the persistentvolume and the persistentvolumeclaim resources in Kubernetes, let's create a pod that can use the volume.

Create a pod named pv-pod that uses the image nginx with a volume named pv-storage . Mount the volume inside the container at /usr/share/nginx/html and specify the pvc by it's name (pv-claim ).

After you've created the pod, list all the pods in the default namespace.

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
    - name: pv-container
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-storage
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
```
