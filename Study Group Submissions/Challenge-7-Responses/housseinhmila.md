**Name**: Houssein Hmila

**Slack User ID**: U054SML63K5

**Email**: housseinhmila@gmail.com   

---

## Solution

### Solution Details

# Kubernetes Cluster Setup Guide

This guide will walk you through the process of setting up a highly available Kubernetes cluster with a control plane using Kubeadm on Microsoft Azure.

## Prerequisites

Before you begin, make sure you've reviewed the [Kubernetes setup prerequisites](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin).

## Infrastructure Setup

The following YAML code demonstrates how to set up the necessary infrastructure on Microsoft Azure:

 ```yaml
 # Create resource group
az group create -n MyKubeCluster -l eastus
# Create VNet
az network vnet create -g MyKubeCluster -n KubeVNet --address-prefix 172.0.0.0/16 \
  --subnet-name MySubnet --subnet-prefix 172.0.0.0/24
# Create Master Node - This will have a public IP
az vm create -n kube-master1 -g MyKubeCluster --image Ubuntu2204 \
  --size Standard_DS2_v2 \
  --data-disk-sizes-gb 10 --generate-ssh-keys \
  --public-ip-address-dns-name kubeadm-master-1
az vm create -n kube-master2 -g MyKubeCluster --image Ubuntu2204 \
  --size Standard_DS2_v2 \
  --data-disk-sizes-gb 10 --generate-ssh-keys \
  --public-ip-address-dns-name kubeadm-master-2
az vm create -n kube-master3 -g MyKubeCluster --image Ubuntu2204 \
  --size Standard_DS2_v2 \
  --data-disk-sizes-gb 10 --generate-ssh-keys \
  --public-ip-address-dns-name kubeadm-master-3
# Create the two worker nodes
az vm create -n kube-worker-1 \
  -g MyKubeCluster --image Ubuntu2204 \
  --size Standard_B1s --data-disk-sizes-gb 4 \
  --generate-ssh-keys 
az vm create -n kube-worker-2 \
  -g MyKubeCluster --image Ubuntu2204 \
  --size Standard_B2s --data-disk-sizes-gb 4 \
  --generate-ssh-keys
 ```
 ### Configuration on All Nodes
 after creating the infra we need to perform these commands on all nodes:
 ```yaml
 sudo apt update
# Install Docker
sudo apt install docker.io -y
sudo systemctl enable docker
# Get the gpg keys for Kubeadm
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
# Install Kubeadm
sudo apt install kubeadm -y
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
#Disable Firewall
ufw disable
#Disable swap
swapoff -a; sed -i '/swap/d' /etc/fstab
#Update sysctl settings for Kubernetes networking
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
 ```
 ### Load Balancer Setup
 now we have to create another vm to play the load balancer role (you can choose any type of vm).
To create a load balancer, follow these steps:
 ```yaml
 sudo apt-get update
sudo apt-get upgrade

# Install HAProxy
sudo apt-get install haproxy

sudo nano /etc/haproxy/haproxy.cfg

# Edit HAProxy configuration
# ... (add HAProxy configuration details)
global
       ....
defaults
       ....
frontend kubernetes-frontend
    bind *:6443
    mode tcp
    option tcplog
    default_backend kubernetes-backend

backend kubernetes-backend
    mode tcp
    option tcp-check
    balance roundrobin
    server kube-master1 <kube-master1-IP>:6443 check fall 3 rise 2
    server kube-master2 <kube-master2-IP>:6443 check fall 3 rise 2
    server kube-master2 <kube-master3-IP>:6443 check fall 3 rise 2
 ```
 ### Don't forget to configure firewall rules to enable communication between the load balancer and master nodes.

you can check that by running 
```ping <loadbalancer-IP>:6443``` inside your master nodes.

### Initializing the Cluster
On one of the master nodes, initialize the cluster:
```
sudo kubeadm init --control-plane-endpoint "<loadbalancer-dns>:6443" --upload-certs
```
If you encounter the "Unimplemented service" error, follow the provided steps to resolve it.
```yaml
sudo vim /etc/crictl.yaml

#put these lines :

runtime-endpoint: "unix:///run/containerd/containerd.sock"
timeout: 0
debug: false
```
After finish the initialization, normally you should see two join commands one for master nodes and the other for worker nodes, execute them to join your rest of nodes.

we can now configure the context so we can perform kubectl as the normal user :(you should run this on all master nodes)
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
the last thing is to deploy the CNI:
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
Once you have completed the setup, you can run the kubectl get node command to view the list of nodes in the cluster. The output of this command should look something like this:
```
NAME                                          STATUS   ROLES    AGE   VERSION
kube-master1                                Ready    master   3m7s   v1.28.2
kube-master2                                Ready    master   3m5s   v1.28.2
kube-master3                                Ready    master   3m3s   v1.28.2
kube-worker1                                Ready    <none>   3m2s   v1.28.2
kube-worker2                                Ready    <none>   3m     v1.28.2
```
### Conclusion
You've successfully set up a highly available Kubernetes cluster with a control plane on Microsoft Azure. Your cluster is now ready for running containerized applications.