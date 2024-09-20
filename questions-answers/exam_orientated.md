
If any kubeapiserver fails
```bash
Log locations to check:

/var/log/pods
/var/log/containers
crictl ps + crictl logs
docker ps + docker logs (in case when Docker is used)
kubelet logs: /var/log/syslog or journalctl


/var/log/pods # nothing
crictl logs # nothing

# syslogs:
tail -f /var/log/syslog | grep apiserver
> Could not process manifest file err="/etc/kubernetes/manifests/kube-apiserver.yaml: couldn't parse as pod(yaml: mapping values are not allowed in this context), please check config file"

# or:
journalctl | grep apiserver
> "Could not process manifest file" err="/etc/kubernetes/manifests/kube-apiserver.yaml: couldnt parse as pod(yaml: mapping values are not allowed in this context), please check config file"

# if you want check details of nodes

kubectl describe node controplane | grep -i taint
```



### Json query

```bash

kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'
master node01

kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}'
amd64 amd64

kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}'
4 4

kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{.items[*].status.capacity.cpu}'
master node01 4 4

kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}' #{"\n"} new line {"\t"} tab

master node01
4       4

'{range.items[*]}

    {.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}

{end}'



'{range.items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'


kubectl get nodes -o=jsonpath='{range.items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'

NODE    CPU
master  4
node01  4


kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
NODE    CPU
master  4
node01  4


kubectl get nodes --sort-by=.status.capacity.cpu #to sort in a specific order.
```


### Last minute commands
```bash

k logs multi-container-pod -c c2 >> /root/log.txt

k -n management get pod -o yaml | grep -i priority -B 20 

#to 

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  nodeSelector:
    kubernetes.io/hostname: controlplane #labels of controlplane node

controlplane $ k get nodes --show-labels   
NAME           STATUS   ROLES           AGE   VERSION   LABELS
controlplane   Ready    control-plane   38d   v1.30.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=controlplane,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
node01         Ready    <none>          38d   v1.30.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux


# to remove taint

kubectl taint no controlplane node-role.kubernetes.io/control-plane:NoSchedule-

# to set context
kubectl config use-context kubernetes-admin@kubernetes


#to store it in specific file
etcdctl snapshot restore /opt/cluster_backup.db --data-dir /root/default.etcd | tee restore.txt

#troubelshooting
initContainers:
  - name: init-container
    image: busybox
    command:
      - sh
      - "-c"
      - echo 'Welcome To KillerCoda!'


#kubelet

location: /var/lib/kubelet/config.yaml and /etc/kubernetes/kubelet.conf

#to get the name of current context

k config get-contexts -o name > /opt/course/1/contexts

#without using kubectl 
cat ~/.kube/config | grep current


#sorting my creationtimestamp in all namespace
kubectl get pod -A --sort-by=.metadata.creationTimestamp

#sort by uid
kubectl get pod -A --sort-by=.metadata.uid

#show resource usage
➜ k top node
NAME               CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
cluster1-controlplane1   178m         8%     1091Mi          57%       
cluster1-node1   66m          6%     834Mi           44%       
cluster1-node2   91m          9%     791Mi           41% 

# to see the top pod  and their containers resource usage
k top pod --container=true

➜ k top pod -h
Display Resource (CPU/Memory/Storage) usage of pods.
...
Namespace in current context is ignored even if specified with --namespace.
      --containers=false: If present, print usage of containers within a pod.
      --no-headers=false: If present, print output without headers.
...


#to show kubelet process

ps aux | grep kubelet


#to stop anything important just remove that outside of the manifests folder.


# to manuvally schedule a pod
  nodeName: cluster2-controlplane1 

role, rolebinding, service account needs namespace



# to schedule in each nodes
      affinity:                                             # add
        podAntiAffinity:                                    # add
          requiredDuringSchedulingIgnoredDuringExecution:   # add
          - labelSelector:                                  # add
              matchExpressions:                             # add
              - key: id                                     # add
                operator: In                                # add
                values:                                     # add
                - very-important                            # add
            topologyKey: kubernetes.io/hostname             # add
status: {}

#another method
        name: container2                # add
      topologySpreadConstraints:                 # add
      - maxSkew: 1                               # add
        topologyKey: kubernetes.io/hostname      # add
        whenUnsatisfiable: DoNotSchedule         # add
        labelSelector:                           # add
          matchLabels:                           # add
            id: very-important                   # add  #this label is very important


#to get the latest events ordered by creationtimestamp
kubectl get events -A --sort-by=.metadata.creationTimestamp


#to run command inside worker nodes
crictl ps #similar to docker ps
crictl rm <container-name> #similar to docker rm
crictl inspect <container-id>  
 root@cluster1-node2:~# crictl inspect b01edbe6f89ed | grep runtimeType
    "runtimeType": "io.containerd.runc.v2",  #info.runtimeType


k get po -n safari --sort-by=.metadata.creationTimestamp | tac
ff
#to list all the resources
k api-resources #shows all

k api-resources -h 

k api-resources --namespaced -o name > /opt/course/16/resources.txt #to see namespace specific resources like pod, seceret, configmap


#to see namespaces with more resources
amespace with most Roles
➜ k -n project-c13 get role --no-headers | wc -l
No resources found in project-c13 namespace.
0

➜ k -n project-c14 get role --no-headers | wc -l
300

➜ k -n project-hamster get role --no-headers | wc -l
No resources found in project-hamster namespace.
0

➜ k -n project-snake get role --no-headers | wc -l
No resources found in project-snake namespace.
0

➜ k -n project-tiger get role --no-headers | wc -l
No 

#to see which nodes it is assigned to
k -n project-tiger get pod tigers-reunite -o jsonpath="{.spec.nodeName}" 


#to find kubelet
whereis kubelet

#to see the version of everything
kubectl version
kubelet --version
 apt show kubectl -a | grep 1.30

#to get service details
➜ k get svc,ep -l run=my-static-pod
NAME                         TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/static-pod-service   NodePort   10.99.168.252   <none>        80:30352/TCP   30s

NAME                           ENDPOINTS      AGE
endpoints/static-pod-service   10.32.0.4:80   30s

#to see certificate validity
➜ root@cluster2-controlplane1:~# openssl x509  -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2
        Validity
            Not Before: Dec 20 18:05:20 2022 GMT
            Not After : Dec 20 18:05:20 2023 GMT


#to check expiration
➜ root@cluster2-controlplane1:~# kubeadm certs check-expiration | grep apiserver
apiserver                Jan 14, 2022 18:49 UTC   363d        ca               no      
apiserver-etcd-client    Jan 14, 2022 18:49 UTC   363d        etcd-ca          no      
apiserver-kubelet-client Jan 14, 2022 18:49 UTC   363d        ca               no 

#to renew the certificates
# /opt/course/22/kubeadm-renew-certs.sh
kubeadm certs renew apiserver


#executing commands
➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.25:1111
database one

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.23:2222
database two

➜ k -n project-snake exec backend-0 -- curl -s 10.44.0.22:3333


#kubectl get secret database-data -n database-ns -o jsonpath="{.data}" > decoded.txt

#name
kubectl get secret database-data -n database-ns -o jsonpath="{.data}" | jq 'to_entries | .[] | {(.key): (.value | @base64d)}' > decoded.txt



 k run output-pod --image=busybox -it --rm --restart=Never -- /bin/sh -c "echo Congratulations! you have passed the CKA exam" > output-pod.txt #temporary pods
```



