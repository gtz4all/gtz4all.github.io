---
title: "Introduction"
date: 2021-06-04T15:48:46-04:00
description: "Using Kubernetes as Docker container management and orchestration."
---

![kubernetes](../../assets/images/kubernetes.png "kubernetes"){: style="width:100px"} [Kubernetes Documentation](/devops/kubernetes/intro)

kubernetes is currently the leading devops provider. 

#### Kubernetes at a Glance
***What is Kubernetes?***
Kubernetes is an open source container management and orchestration system which provides built in cloud navite features:
* Managing clusters of containers
* Providing tools for deploying applications
* Auto Scaling applications as and when needed
* Managing changes to the existing containerized applications
* Helping to optimize the use of underlying hardware beneath your container
* Enableing the application component to restart and move across the system as and when needed

#### Kubernetes Architecture and Componets
Kubernetes includes ```multiple components``` to manage and control the cluster:
***Backend System Pods***
```bash
root@f8aca9a6c11c:/home/cloud_user# kubectl get pods -n kube-system
NAME                                                   READY   STATUS    RESTARTS   AGE
coredns-584795fc57-qd468                               1/1     Running   0          25m
coredns-584795fc57-whf8j                               1/1     Running   0          25m
etcd-f8aca9a6c11c.mylabserver.com                      1/1     Running   0          25m
kube-apiserver-f8aca9a6c11c.mylabserver.com            1/1     Running   0          25m
kube-controller-manager-f8aca9a6c11c.mylabserver.com   1/1     Running   0          24m
kube-flannel-ds-amd64-dk4bm                            1/1     Running   0          10m
kube-flannel-ds-amd64-rbz6w                            1/1     Running   0          10m
kube-flannel-ds-amd64-xgvnv                            1/1     Running   0          10m
kube-proxy-hc6xr                                       1/1     Running   0          19m
kube-proxy-hlmlk                                       1/1     Running   0          25m
kube-proxy-wpmhm                                       1/1     Running   0          20m
kube-scheduler-f8aca9a6c11c.mylabserver.com            1/1     Running   0          25m
```

###### Kube Master Control Plane
* ***etcd***: provides distrubuted synchronized data storage for the cluster state. Information about the cluster such as pods, nodes, etc are stored here.

* ***kube-apiserver***: Primary interface for the cluster which is a simple REST based web API. ```kubectl``` this API to interact with the cluster.

* ***kube-controller-manager***: bundles several backend components doing all the behind the scenes work of controlling the cluster.

* ***kube-scheduler***: determines when to run pods and what nodes to run them on based on deployments or other automation

###### Individual Kube Node Components
* ***kubelet***: agent used to interact between the kubernetes's API and contanier's runtime. Masters control place communicates to node kubelets which instruct docker to create/run the container. Master and worker nodes have this agent installed

***kubelet runs as a service***
```bash
root@f8aca9a6c11c:/home/cloud_user# sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Sun 2021-07-11 13:41:35 UTC; 1h 18min ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 7174 (kubelet)
    Tasks: 17 (limit: 2312)
   CGroup: /system.slice/kubelet.service
           └─7174 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/conf

Jul 11 13:56:51 f8aca9a6c11c.mylabserver.com kubelet[7174]: E0711 13:56:51.107949    7174 kubelet.go:2170] Container runtime network not ready: NetworkReady=false reason
Jul 11 13:56:55 f8aca9a6c11c.mylabserver.com kubelet[7174]: W0711 13:56:55.713797    7174 cni.go:213] Unable to update cni config: No networks found in /etc/cni/net.d
Jul 11 13:56:56 f8aca9a6c11c.mylabserver.com kubelet[7174]: E0711 13:56:56.109215    7174 kubelet.go:2170] Container runtime network not ready: NetworkReady=false reason
Jul 11 13:56:59 f8aca9a6c11c.mylabserver.com kubelet[7174]: E0711 13:56:59.901445    7174 reflector.go:126] object-"kube-system"/"flannel-token-sxgpr": Failed to list *v
Jul 11 13:57:00 f8aca9a6c11c.mylabserver.com kubelet[7174]: I0711 13:57:00.049225    7174 reconciler.go:207] operationExecutor.VerifyControllerAttachedVolume started for
Jul 11 13:57:00 f8aca9a6c11c.mylabserver.com kubelet[7174]: I0711 13:57:00.052181    7174 reconciler.go:207] operationExecutor.VerifyControllerAttachedVolume started for
Jul 11 13:57:00 f8aca9a6c11c.mylabserver.com kubelet[7174]: I0711 13:57:00.052995    7174 reconciler.go:207] operationExecutor.VerifyControllerAttachedVolume started for
Jul 11 13:57:00 f8aca9a6c11c.mylabserver.com kubelet[7174]: I0711 13:57:00.058499    7174 reconciler.go:207] operationExecutor.VerifyControllerAttachedVolume started for
Jul 11 13:57:00 f8aca9a6c11c.mylabserver.com kubelet[7174]: W0711 13:57:00.714022    7174 cni.go:213] Unable to update cni config: No networks found in /etc/cni/net.d
Jul 11 13:57:01 f8aca9a6c11c.mylabserver.com kubelet[7174]: E0711 13:57:01.110293    7174 kubelet.go:2170] Container runtime network not ready: NetworkReady=false reason
r
```
*  ****kube-proxy***: each nodes ( master and workers ) needs its own, handles virtual network communication between nodes by adding firewall routing rules when pods are trying to communicate across nodes.


