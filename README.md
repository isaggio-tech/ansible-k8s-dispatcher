+++ Remove A Node (Worker Node)

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

Join the node again as a fresh node


+++ Readd Cordened Node:
  kubectl uncordon <node name>

########################################

Usage:

Modify Files, hosts | env_variables | ansible.cfg (Adjust to your env details ip's, private_key_file etc) 

ansible-playbook deploy_k8_master.yml -e "hostGroup=KubeMasters"
ansible-playbook deploy_k8_workers.yml -e "hostGroup=KubeWorkers"

APPENDIX: Clean Installation

systemctl stop docker kubelet
systemctl disable docker kubelet
yum remove docker* kube* -y

Find and Remove any Folders/Files for docker and Kubernetes/kubelet

find / -name 'docker'
find / -name 'kubernetes*'
find / -name 'kubelet'
