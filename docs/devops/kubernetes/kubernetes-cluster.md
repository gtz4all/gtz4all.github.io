---
title: "Kubernetes Cluster"
date: 2021-07-14T11:29:58-04:00
draft: false
description: "Creating a Kubernetes Cluster using kubeadm"
---

#### Creating a Kubernetes Cluster using Kubeadm
***Kubeadm*** is a tool to easily bootstrap a kubernetes cluster with the minimun requirements.

* ***Info*** ```all nodes``` includes nodes and master nodes.

<div class="mermaid">
graph TD
  subgraph "Kubernetes Cluster"
  A{Master<br/>Node} -->|Node1|B(Node One)
  A{Master<br/>Node} -->|Node2|C(Node Two)
  A{Master<br/>Node} -->|Node3|D(Node Three)
  end
</div>
<script async src="https://unpkg.com/mermaid@8.2.3/dist/mermaid.min.js"></script>

#### Requirements
* Instance - Ubuntu 18.04 Bionic Beaver LTS
* Docker
* Kubeadm, Kubelet, and Kubectl

#### Docker install
* 
1. Install Docker on all nodes including master node:
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
sudo apt-mark hold docker-ce
```

2. if running any other linux flavor use the convenience script provided by docker.
```bash
 ~$ curl -fsSL https://get.docker.com -o get-docker.sh
 ~$ sudo sh get-docker.sh
 ```
 
3. Verify that Docker is up and running with:
```bash
~$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2021-07-16 19:47:37 UTC; 3h 49min ago
     Docs: https://docs.docker.com
 Main PID: 909 (dockerd)

```
Make sure the Docker service status is ```active (running)```!

```bash
~$ sudo docker version
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:24:51 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       e68fc7a
  Built:            Tue Aug 21 17:23:15 2018
  OS/Arch:          linux/amd64
  Experimental:     false
```

#### Install Kubeadm, Kubelet, and Kubectl on all nodes.
* kubeadm: cluster bootstrapper.
* kubelet: creates pods and containers, run on all members of the cluster.
* kubectl: communicates with cluster's API.

1. Install the Kubernetes components by running this on all nodes:
```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet=1.14.5-00 kubeadm=1.14.5-00 kubectl=1.14.5-00
sudo apt-mark hold kubelet kubeadm kubectl
```

#### Bootstrap the cluster on the Kube master node.
1. On the Kube master node, do this:
```bash
sudo kubeadm init --pod-network-cidr=10.10.0.0/16
```
That command may take a few minutes to complete. 
***output***
```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.109.72:6443 --token ku77lq.2aupucmcqy3mmjuz \
    --discovery-token-ca-cert-hash sha256:533e64c88c12b7f40b8bf2283ac15dc1b080003ad92c8f53e1742c86e94d6f4c
```


2. running kubectl as non-root user:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
* optionally
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Take note that the ```kubeadm init``` command printed a long ```kubeadm join``` command to the screen. You will need that ```kubeadm join``` command in the next step!

2. Run the following commmand on the Kube master node to verify it is up and running:
```bash
kubectl version
```
This command should return both a ```Client Version``` and a ```Server Version```.


#### Join the two Kube worker nodes to the cluster.
1. Copy the ```kubeadm join``` command that was printed by the ```kubeadm init``` command earlier, with the token and hash. Run this command on both worker nodes, but make sure you add ```sudo``` in front of it:
```bash
sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash
```
2. Now, on the Kube master node, make sure your nodes joined the cluster successfully:
```bash
kubectl get nodes
```
Verify that all of your nodes are listed. It will look something like this:
```bash
NAME            STATUS     ROLES    AGE   VERSION
ip-10-0-1-101   NotReady   master   30s   v1.12.2
ip-10-0-1-102   NotReady   &lt;none>   8s    v1.12.2
ip-10-0-1-103   NotReady   &lt;none>   5s    v1.12.2
```
Note that the nodes are expected to be in the NotReady state for now.


#### Set up cluster networking with flannel.
1. Turn on iptables bridge calls on all nodes:
```bash
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```
2. Next, run this only on the Kube master node:
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```
Now flannel is installed! Make sure it is working by checking the node status again:
```bash
kubectl get nodes
```
After a short time, all nodes should be in the ```Ready``` state. If they are not all ```Ready``` the first time you run ```kubectl get nodes```, wait a few moments and try again. It should look something like this:
```bash
NAME            STATUS   ROLES    AGE   VERSION
ip-10-0-1-101   Ready    master   85s   v1.12.2
ip-10-0-1-102   Ready    &lt;none>   63s   v1.12.2
ip-10-0-1-103   Ready    &lt;none>   60s   v1.12.2
```
