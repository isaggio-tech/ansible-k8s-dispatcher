# Ansible K8s Dispatcher

Anisble K8s Dispatcher helps in deploying K8s full cluster using ansible playbook, currently it supports CentOs/Redhat but can be easily tweaked to make it work for other OS
The Playbooks are tested on CentOS 6 & 7 Versions both for Physical Servers and Kernel VM's

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

Should have below installed on the Ansible playbook's executing Host

```
ansible
```

Should have below on K8s cluster Nodes,

```
> Static IP's Configured
> Passwordless access from the Ansible Managing host to all K8s Nodes
> private_key_file in the right Location - to be used inside anisble.cfg
```

### Installing/Getting Ready

Git Clone this repo and modify and update the below files,

```
hosts
env_variables
ansible.cfg (remote_user, private_key_file etc)
```

### Run Playbook

Running playbook is simple and we have 2 wrapper playbooks that does the work for us,
First lets test the connection to the cluster nodes,


## Test Connection

echo hello on all nodes

```
ansible all -a "/bin/echo hello"

k8s-worker-vm1 | CHANGED | rc=0 >>
hello
k8s-worker-vm2 | CHANGED | rc=0 >>
hello
k8s-worker-vm3 | CHANGED | rc=0 >>
hello
k8s-master | CHANGED | rc=0 >>
hello

```

If you see above output, please proceed deploying the playbook

## Deploy Master Node

Assuming you have defined k8s-master with the correct IP address, run below

```
ansible-playbook deploy_k8_master.yml -e "hostGroup=KubeMasters"
```

## Deploy Worker Nodes

Assuming you have defined k8s-worker-xx with the correct IP address, 

If the deployment is for the first time use below,

```
ansible-playbook deploy_k8_workers.yml -e "hostGroup=KubeWorkers redeploy=false"
```

If the deployment is not new and you want to re-deploy the worker nodes. use below,

Note: This will drain and delete and re-deploy worker nodes - so please be careful with your deployments

```
ansible-playbook deploy_k8_workers.yml -e "hostGroup=KubeWorkers redeploy=true"
```

Note: Adding new worker node also requires redeployment flag

### Verify Cluster Status

Connect to K8s master Node, and run below

```
kubectl cluster-info
Kubernetes master is running at https://192.168.1.28:6443
KubeDNS is running at https://192.168.1.28:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

```
[root@tech ~]# kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
centos7-vm1.isaggio.com   Ready    <none>   17h   v1.18.0
centos7-vm2.isaggio.com   Ready    <none>   17h   v1.18.0
centos7-vm3.isaggio.com   Ready    <none>   17h   v1.18.0
tech.isaggio.com          Ready    master   17h   v1.18.0
```

All good, you have working Cluster

### Troubleshooting

Error 1:

```
ERROR:
ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1"

kubeadm reset

echo 1 > /proc/sys/net/ipv4/ip_forward
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables

[root@tech ~]# cat /proc/sys/net/bridge/bridge-nf-call-iptables
1
[root@tech ~]# cat /proc/sys/net/ipv4/ip_forward
1
```

## Extensions

I found cockpit is a very good application if you want to visualize the K8s deployments and placements, 
We will create a another repo to cover that through ansible setup


## Built With

* [ansible](https://opensource.com/article/18/7/sysadmin-tasks-ansible) - IaaC Tool

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details


### Appendix

##Remove A Node (Worker Node)

```
Cordon the node
kubectl cordon <node name>

Drain the node
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets

Delete the node
kubectl delete node <node name>

Reset the node ( run kubeadm reset command if it is joined using kubeadm)
Run on the node being removed, reset all kubeadm installed state:

kubeadm reset

If you want to reset the IPVS tables, you must run the following command:
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

##Join the node again as a fresh node

```
kubectl uncordon <node name>
```

##Clean K8s Nodes

Remove docker and kubelet Packages,

```
systemctl stop docker kubelet; systemctl disable docker kubelet; yum remove docker* kube* -y

Find and Remove any Folders/Files for docker and Kubernetes/kubelet:

```
find / -name 'docker' | awk '{print "rm -rf " $0}' ; find / -name 'kubernetes*' | awk '{print "rm -rf " $0}'; find / -name 'kubelet' | awk '{print "rm -rf " $0}'
```

Clean etcd on Master:

```
rm -rf /var/lib/etcd
```
