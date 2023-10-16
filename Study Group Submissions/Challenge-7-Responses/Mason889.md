**Name**: Kostiantyn Zaihraiev

**Slack User ID**: U04FZK4HL0L

**Email**: kostya.zaigraev@gmail.com

---

## Solution

### Pre-requisites

Have AWS account.

### Solution Details

#### Preparations

After reading task, I've decided to create 3 virtual machines in AWS.

And setup AWS Network Load Balancer (NLB) with 3 targets (one per control plane node).

I've opened such ports for NLB:
* 6443 - for control plane nodes
* 2379-2380 - for etcd
* 10250 - for kubelet API
* 10251 - for kube-scheduler
* 10252 - for kube-controller-manager

I've spinned 3 EC2 instances based on Ubuntu images with names `control-plane-1`, `control-plane-2`, `control-plane-3` and kubeadm, kubelet and kubectl using commands below.

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo sh -c 'echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list'
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl containerd
```

After that I've disabled swap on all nodes:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Make sure that sysctl parameters `net.bridge.bridge-nf-call-iptables` and `net.ipv4.ip_forward` are set to `1` on all nodes:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
sudo sysctl net.ipv4.ip_forward=1
```

#### Control-Planes initialization

On the `control-plane-1` node I've initialized the cluster:

```bash
sudo kubeadm init --control-plane-endpoint <NLB DNS record>:6443 --upload-certs --kubernetes-version 1.28.0
```

In the output of the command I've found the command to join other control plane nodes to the cluster:

```bash
kubeadm join <NLB DNS record>:6443 --token e2ya3q.qn6om3tbo4pwvspl \
	--discovery-token-ca-cert-hash sha256:23528857afadce668be595d8954995257f27f1d984001f025cceec1fdd99071f \
	--control-plane --certificate-key 3200e4d214ccbf384bc5f1e95e9eeea40035d96af499ac82b3a531965b8209c7
```

I've executed the command on the `control-plane-2` and `control-plane-3` nodes.

After that I've installed the CNI plugin:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

While I've waited for the command execution I've created `.kube` directory and copied `admin.conf` file to it (these steps were done on the `control-plane-1` node):

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

After that I've applied the following command on all control plane nodes to apply the CNI plugin:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

And after that I've checked that all nodes are in `Ready` state:

```bash
kubectl get node

NAME              STATUS     ROLES           AGE     VERSION
ip-172-16-1-110   Ready      control-plane   10m2s   v1.28.2
ip-172-16-1-56    Ready      control-plane   6m12s   v1.28.2
ip-172-16-1-74    Ready      control-plane   6m14s   v1.28.2
```

#### Worker node initialization

Required to do prerequisite steps on the worker node before joining it to the cluster (described in section upper)

I've created one worker node using the following command:

```bash
kubeadm join <NLB DNS record>:6443 --token e2ya3q.qn6om3tbo4pwvspl \
	--discovery-token-ca-cert-hash sha256:23528857afadce668be595d8954995257f27f1d984001f025cceec1fdd99071f
```

After that I've applied the following command on worker node to apply the CNI plugin:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

And after that I've checked that all nodes are in `Ready` state (done on the `control-plane-1` node):

```bash
kubectl get node


NAME              STATUS     ROLES           AGE      VERSION
ip-172-16-1-110   Ready      control-plane   14m52s   v1.28.2
ip-172-16-1-56    Ready      control-plane   10m31s   v1.28.2
ip-172-16-1-74    Ready      control-plane   10m29s   v1.28.2
ip-172-16-1-237   Ready      <none>          58s      v1.28.2
```

### Code Snippet
