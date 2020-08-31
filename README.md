# App deployment

This repo aims to deploy a simple App that cathes the IP and hostname where It is running and shows It on a web server. Minikube has been used.


## App deployment using Docker-Compose

Inside /docker folder, Dockerfile and Python file are written to compile the App image which will be used in Docker and K8s. Steps to reproduce:

1. Build image from Dockerfile:

```
$ sudo docker build -t app-local-ip:0.1 .
```

2. Run a container based on that image:

```
$ sudo docker run -h myHostnameIBM --publish 80:80 --detach --name myapp app-local-ip:0.1
```

## App deploument using K8s

Inside /k8s folder, yaml files are written to create a new deployment and a new service:

Steps to reproduce:

1. Create deployment and test It is running:

```
$ kubectl apply -f app-deployment.yaml
$ kubectl get pods -n test
NAME                      READY   STATUS    RESTARTS   AGE
ip-app-7f45789ff8-vcbwk   1/1     Running   0          42m
```

2. Scale if requested:

```
$ kubectl scale --replicas=3 -f app-deployment.yaml 
deployment.apps/ip-app scaled
$ kubectl get rs -n test
NAME                DESIRED   CURRENT   READY   AGE
ip-app-7f45789ff8   3         3         1       5m22s

```

Apart from this way, It is possible to change spec/replicas in app-deployment.yaml obtaining the same result.

3. Deploy a server to access it from outside the cluster

```
$ kubectl apply -f app-service.yaml 
```

Service is running on pod IP and port 80 (direct connection to App).
```
$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ curl 172.17.0.2
Hostname :   ip-app-deployment-fd864489-9hmf2
IP :  172.17.0.2
```
In order to see endpoints and check the service running using nodePort and assuming 192.168.99.100 as Minikube node port (extracted with "kubectl describe node minikube"):
```
$ kubectl get svc -n test
NAME     TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
ip-app   NodePort   10.99.15.254   <none>        8080:32695/TCP   19s
$ kubectl describe svc ip-app -n test
Name:                     ip-app
Namespace:                test
Labels:                   run=ip-app
Annotations:              <none>
Selector:                 app=ip-app
Type:                     NodePort
IP:                       10.99.15.254
Port:                     <unset>  8080/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32695/TCP
Endpoints:                172.17.0.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
$ curl 192.168.99.100:32695
Hostname :   ip-app-fd864489-jjvp6
IP :  172.17.0.2
```