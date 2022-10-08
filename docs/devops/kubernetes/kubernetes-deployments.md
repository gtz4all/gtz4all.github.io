---
title: "Kubernetes Deployments"
date: 2021-06-04T11:29:58-04:00
draft: false
description: "Deployments provide pod scaling, rolling-updates, and self-healing services right out of the box"
---
#### Kubernetes Deployments
***Deployments*** provide a way to spin up and automate multiple pods by specifying a ``desired state`` to be maintained by the cluster.
* ***Scaling*** the deployment will either create or delete pods in order to meet the number of replicas requested. The deployment can dynamically adjust itself without impacting appliaction.
* ***Rolling Updates***: the deployment can be adjusted to a new image version. it will gradually spin up new containers with the new vesrion to replace existing containers.
* ***Self-Healing***: if any of pods in the deployment is destroyed, the deployment will create new ones in order to meet desired number of replicas.

#### deployment
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-app

spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.1
        ports:
        - containerPort: 80
```
use ```kubectl create -f <deployment-file.yml>``` to build deployment.
```bash
# kubectl create -f ./kube-deployment.yml
deployment.apps/nginx-deployment created
# kubectl get deployment
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           9s
# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6fb8795c7d-dq9zp   1/1     Running   0          22s
nginx-deployment-6fb8795c7d-r8bwk   1/1     Running   0          22s
```
the ```kubectl describe deployment <deployment-name>``` provides detailed information about the deployment. the event section provides any deployment changes such scaling, self-healing, and rolling-updates

```bash
# kubectl describe deployment nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 15 Jul 2021 23:57:20 +0000
Labels:                 app=nginx-app
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx-app
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx-app
  Containers:
   nginx:
    Image:        nginx:1.20.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-7d569cb5f5 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  16s   deployment-controller  Scaled up replica set nginx-deployment-7d569cb5f5 to 2
  ```
#### Deployment Scaling
the deployment will either create or delete pods in order to meet the number of replicas requested. The deployment can dynamically adjust itself without impacting appliaction.
###### Scale up
* Edit the ```deployment.yml``` file spec: replicas from ```2``` to ```6```. Optionally, we can edit the deployment running-config using ```kubectl edit deployment.v1.apps/nginx-deployment```
```yml
##omitted sections
spec:
  replicas: 6
```
* apply changes
```bash
# kubectl apply -f kube-deployment.yml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/nginx-deployment configured
```
* Optionally, we can edit the deployment running-config using ```kubectl edit deployment.v1.apps/nginx-deployment``` use ```i``` to edit and ```[ESC]``` followed by ```:wq!``` to apply changes


> we can do it with a single command ```kubectl scale deployments/nginx-deployment --replicas=6```


```
# kubectl edit deployment.v1.apps/nginx-deployment
deployment.apps/nginx-deployment edited
```
* Validating
```bash
# kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-7d569cb5f5-5kc9r   0/1     ContainerCreating   0          2s
nginx-deployment-7d569cb5f5-cqdqg   0/1     ContainerCreating   0          2s
nginx-deployment-7d569cb5f5-h7x79   1/1     Running             0          26m
nginx-deployment-7d569cb5f5-l2pbh   0/1     ContainerCreating   0          2s
nginx-deployment-7d569cb5f5-r2ln4   1/1     Running             0          26m
nginx-deployment-7d569cb5f5-wtbsw   0/1     ContainerCreating   0          2s
# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7d569cb5f5-5kc9r   1/1     Running   0          4s
nginx-deployment-7d569cb5f5-cqdqg   1/1     Running   0          4s
nginx-deployment-7d569cb5f5-h7x79   1/1     Running   0          26m
nginx-deployment-7d569cb5f5-l2pbh   1/1     Running   0          4s
nginx-deployment-7d569cb5f5-r2ln4   1/1     Running   0          26m
nginx-deployment-7d569cb5f5-wtbsw   1/1     Running   0          4s
```

###### Scale Down
update ```kubectl edit deployment.v1.apps/nginx-deployment``` replicas from ```6``` to ```2```
```yml
# kubectl get pods
NAME                                READY   STATUS        RESTARTS   AGE
nginx-deployment-7d569cb5f5-5kc9r   0/1     Terminating   0          2m15s
nginx-deployment-7d569cb5f5-cqdqg   1/1     Running       0          2m15s
nginx-deployment-7d569cb5f5-h7x79   1/1     Running       0          28m
nginx-deployment-7d569cb5f5-l2pbh   1/1     Running       0          2m15s
nginx-deployment-7d569cb5f5-r2ln4   1/1     Running       0          28m
nginx-deployment-7d569cb5f5-wtbsw   0/1     Terminating   0          2m15s
# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7d569cb5f5-cqdqg   1/1     Running   0          2m20s
nginx-deployment-7d569cb5f5-h7x79   1/1     Running   0          28m
nginx-deployment-7d569cb5f5-l2pbh   1/1     Running   0          2m20s
nginx-deployment-7d569cb5f5-r2ln4   1/1     Running   0          28m
```
###### Scaling Loggs
```bash
# kubectl describe deployment nginx-deployment
### ommitted output ###
Events:
  Type    Reason             Age                  From                   Message
  ----    ------             ----                 ----                   -------
  Normal  ScalingReplicaSet  30m                  deployment-controller  Scaled up replica set nginx-deployment-7d569cb5f5 to 2
  Normal  ScalingReplicaSet  7m12s                deployment-controller  Scaled down replica set nginx-deployment-7d569cb5f5 to 2
  Normal  ScalingReplicaSet  3m25s (x3 over 16m)  deployment-controller  Scaled up replica set nginx-deployment-7d569cb5f5 to 6
  Normal  ScalingReplicaSet  72s (x2 over 11m)    deployment-controller  Scaled down replica set nginx-deployment-7d569cb5f5 to 4
```
#### Deployment Self-Healing
if any of pods in the deployment is destroyed, the deployment will create new ones in order to meet desired number of replicas

```bash
# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7d569cb5f5-cqdqg   1/1     Running   0          8m
nginx-deployment-7d569cb5f5-l2pbh   1/1     Running   0          8m
nginx-deployment-7d569cb5f5-nbb4w   1/1     Running   0          4s
nginx-deployment-7d569cb5f5-r2ln4   1/1     Running   0          34m
# kubectl delete pod nginx-deployment-7d569cb5f5-r2ln4
pod "nginx-deployment-7d569cb5f5-r2ln4" deleted
# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7d569cb5f5-cqdqg   1/1     Running   0          8m38s
nginx-deployment-7d569cb5f5-l2pbh   1/1     Running   0          8m38s
nginx-deployment-7d569cb5f5-nbb4w   1/1     Running   0          42s
nginx-deployment-7d569cb5f5-tjsnw   1/1     Running   0          3s
```
the cluster build a new container right after I requested the deletion immediately. I couldnt catch the cluster creating the new container but we can tell by looking at the AGE value. ```nginx-deployment-7d569cb5f5-tjsnw``` was created when I deleted ```nginx-deployment-7d569cb5f5-r2ln4```.

#### Deployment Rolling Updates
the deployment can be adjusted to a new image version. it will gradually spin up new containers with the new vesrion to replace existing containers.
Upgrading from ```image: nginx:1.20.1``` to ```nginx=nginx:1.21.1```. The cluster will create new containers with the new image once those containers are running it will terminate the containers running the old image:
```bash
# kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-5cfcccdb74-ck2nj   1/1     Running             0          7s
nginx-deployment-5cfcccdb74-lvgq2   0/1     ContainerCreating   0          2s
nginx-deployment-5cfcccdb74-v2k4g   0/1     ContainerCreating   0          1s
nginx-deployment-5cfcccdb74-zn666   1/1     Running             0          7s
nginx-deployment-7d569cb5f5-cqdqg   1/1     Terminating         0          17m
nginx-deployment-7d569cb5f5-l2pbh   1/1     Running             0          17m
nginx-deployment-7d569cb5f5-nbb4w   0/1     Terminating         0          9m29s
```
#### Conclusion 
Edittig the running-config vs editing the actual file and apply the changes works the same way. 
