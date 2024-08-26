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