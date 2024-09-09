
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


kubectl get nodes --sort-by=.status.capacity.cp #to sort in a specific order.
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
```



