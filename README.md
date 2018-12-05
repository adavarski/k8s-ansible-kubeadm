### Vagrant ENV:

Run `vagrant up`. This will automatically provision two VM with Kubernetes cluster installed.

```
k8s Environment Info:
Kubernetes version: 1.13 (or latest from repo)
CNI: Weave-Net
Default number of nodes: 2

root@k8s-m1:~# export KUBECONFIG=/etc/kubernetes/admin.conf
root@k8s-m1:~# kubectl get all --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-86c58d9df4-4tg5l                1/1     Running   0          36m
kube-system   pod/coredns-86c58d9df4-jjdjr                1/1     Running   0          36m
kube-system   pod/etcd-k8s-m1                             1/1     Running   0          52m
kube-system   pod/kube-apiserver-k8s-m1                   1/1     Running   0          52m
kube-system   pod/kube-controller-manager-k8s-m1          1/1     Running   3          52m
kube-system   pod/kube-proxy-cbw8d                        1/1     Running   0          52m
kube-system   pod/kube-proxy-cncbd                        1/1     Running   0          44m
kube-system   pod/kube-scheduler-k8s-m1                   1/1     Running   2          52m
kube-system   pod/kubernetes-dashboard-79ff88449c-pqgx8   1/1     Running   0          52m
kube-system   pod/weave-net-ktbvl                         2/2     Running   0          51m
kube-system   pod/weave-net-qzfcx                         2/2     Running   3          44m

NAMESPACE     NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP         52m
kube-system   service/kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP   52m
kube-system   service/kubernetes-dashboard   ClusterIP   10.111.210.147   <none>        443/TCP         52m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/kube-proxy   2         2         2       2            2           <none>          52m
kube-system   daemonset.apps/weave-net    2         2         2       2            2           <none>          51m

NAMESPACE     NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns                2/2     2            2           52m
kube-system   deployment.apps/kubernetes-dashboard   1/1     1            1           52m

NAMESPACE     NAME                                              DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-86c58d9df4                2         2         2       52m
kube-system   replicaset.apps/kubernetes-dashboard-79ff88449c   1         1         1       52m


root@k8s-m1:~# kubectl get pods -o wide --sort-by="{.spec.nodeName}" --all-namespaces
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-86c58d9df4-jjdjr                1/1     Running   0          38m   10.32.0.3      k8s-m1   <none>           <none>
kube-system   coredns-86c58d9df4-4tg5l                1/1     Running   0          38m   10.36.0.0      k8s-n1   <none>           <none>
kube-system   etcd-k8s-m1                             1/1     Running   0          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-apiserver-k8s-m1                   1/1     Running   0          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-controller-manager-k8s-m1          1/1     Running   3          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-proxy-cbw8d                        1/1     Running   0          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-scheduler-k8s-m1                   1/1     Running   2          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-proxy-cncbd                        1/1     Running   0          45m   192.16.35.10   k8s-n1   <none>           <none>
kube-system   kubernetes-dashboard-79ff88449c-pqgx8   1/1     Running   0          53m   10.32.0.2      k8s-m1   <none>           <none>
kube-system   weave-net-ktbvl                         2/2     Running   0          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   weave-net-qzfcx                         2/2     Running   3          45m   192.16.35.10   k8s-n1   <none>           <none>
root@k8s-m1:~# 


```
You can edit Vagrantfile and hack/setup-vms.sh for your needs

Example:
```
Vagrantfile: 
  master = 1
  node = 2

$ diff setup-vms.sh setup-vms.sh.NEW 
11d10
< 
13,14c12,13
< 192.16.35.11 k8s-m1
< 
---
> 192.16.35.11 k8s-n2
> 192.16.35.12 k8s-m1
38c37
<   HOSTS="192.16.35.10 192.16.35.11"
---
>   HOSTS="192.16.35.10 192.16.35.11 192.16.35.12"
```

## Manual Installation

### Kubeadm Ansible Playbook

Build a Kubernetes cluster using Ansible with kubeadm. The goal is easily install a Kubernetes cluster on machines running:

  - Ubuntu 16.04
  - CentOS 7
  - Debian 9

System requirements:

  - Deployment environment must have Ansible `2.4.0+`
  - Master and nodes must have passwordless SSH access

### Usage

Add the system information gathered above into a file called `hosts.ini`. For example:
```
[master]
192.16.35.12

[node]
192.16.35.[10:11]

[kube-cluster:children]
master
node
```

Before continuing, edit `group_vars/all.yml` to your specified configuration.

**Note:**  By default, `k8s-ansible-kubeadm` uses `eth1`. Your default interface may be `eth0`.

After going through the setup, run the `site.yaml` playbook:

```sh
$ ansible-playbook site.yaml
...
==> master1: TASK [addon : Create Kubernetes dashboard deployment] **************************
==> master1: changed: [192.16.35.12 -> 192.16.35.12]
==> master1:
==> master1: PLAY RECAP *********************************************************************
==> master1: 192.16.35.10               : ok=18   changed=14   unreachable=0    failed=0
==> master1: 192.16.35.11               : ok=18   changed=14   unreachable=0    failed=0
==> master1: 192.16.35.12               : ok=34   changed=29   unreachable=0    failed=0
```

Download the `admin.conf` from the master node:

```sh
$ scp k8s@k8s-master:/etc/kubernetes/admin.conf .
```

Verify cluster is fully running using kubectl:

```sh

$ export KUBECONFIG=~/admin.conf
$ kubectl get node
NAME      STATUS    AGE       VERSION
master1   Ready     22m       v1.6.3
node1     Ready     20m       v1.6.3
node2     Ready     20m       v1.6.3

$ kubectl get po -n kube-system
NAME                                    READY     STATUS    RESTARTS   AGE
etcd-master1                            1/1       Running   0          23m
...
```

### Resetting the environment

Finally, reset all kubeadm installed state using `reset-site.yaml` playbook:

```sh
$ ansible-playbook reset-site.yaml
```
