---
title: "Kubernetes Services"
date: 2021-06-04T11:29:58-04:00
draft: false
description: "Deployments provide pod abstraction and load balancing services ensuring traffic is routed towards running pods"
---

#### Kubernetes Services
Kubernetes Services provides an abstraction layer to expose pods created by the deployment. Duo to the pod dynamic nature where pods are dynamically destroyed and created by scaling up/down or rolling updates. Services load balance client connections making sure traffic is only routed to active/running pods that are part of the the service

<div class="mermaid">
graph LR
  subgraph "Kubernetes Cluster"
  Kubernetes_Service -->B{nginx-deployment}
  B -->|Pod1|E[nginx-deployment-5cfcccdb74-ck2nj  ]
  B -->|Pod2|E.A[nginx-deployment-5cfcccdb74-lvgq2]
  B -->|Pod3|E.B[nginx-deployment-5cfcccdb74-v2k4g]
  B -->|Pod4|E.C[nginx-deployment-5cfcccdb74-zn666]
end
</div>
<script async src="https://unpkg.com/mermaid@8.2.3/dist/mermaid.min.js"></script>

```yml
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```
validate service is running

```bash
$# kubectl create -f kube-service.yml
service/nginx-service created

$# kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        5d8h
nginx-service   NodePort    10.100.81.26   <none>        80:32180/TCP   5s

$# curl localhost:32180
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
#### Validating
```kubectl describe svc``` can provide an Endpoint list of all running pods

```bash
# kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        5d8h
nginx-service   NodePort    10.100.81.26   <none>        80:32180/TCP   19m
root@f8aca9a6c11c:/home/cloud_user# kubectl describe svc nginx-service
Name:                     nginx-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx-app
Type:                     NodePort
IP:                       10.100.81.26
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32180/TCP
Endpoints:                10.244.1.20:80,10.244.1.22:80,10.244.2.14:80 + 1 more..
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
In order to validate only running pods are in the Endpoint list, scale down the deployment:
* current state

```bash
# kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5cfcccdb74-ck2nj   1/1     Running   2          21h
nginx-deployment-5cfcccdb74-lvgq2   1/1     Running   2          21h
nginx-deployment-5cfcccdb74-v2k4g   1/1     Running   2          21h
nginx-deployment-5cfcccdb74-zn666   1/1     Running   2          21h
```

* use ```kubectl scale deployments/nginx-deployment --replicas=2``` to scale down from 4 to 2
```bash
# kubectl scale deployments/nginx-deployment --replicas=2
deployment.extensions/nginx-deployment scaled
# kubectl get pods
NAME                                READY   STATUS        RESTARTS   AGE
nginx-deployment-5cfcccdb74-ck2nj   1/1     Running       2          21h
nginx-deployment-5cfcccdb74-lvgq2   1/1     Terminating   2          21h
nginx-deployment-5cfcccdb74-v2k4g   1/1     Running       2          21h
nginx-deployment-5cfcccdb74-zn666   0/1     Terminating   2          21h
# kubectl describe svc nginx-service
Name:                     nginx-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx-app
Type:                     NodePort
IP:                       10.100.81.26
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32180/TCP
Endpoints:                10.244.2.14:80,10.244.2.15:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
