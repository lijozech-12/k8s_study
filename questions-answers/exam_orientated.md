
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



