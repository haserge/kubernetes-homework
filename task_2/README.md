# Task 2
### Create deployment with simple application
```bash
kubectl apply -f nginx-configmap.yaml
kubectl apply -f deployment.yaml
```
### Get pod ip address
```bash
kubectl get pods -o wide
```
### My output
```bash
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
web-6745ffd5c8-7lgf2   1/1     Running   0          28m   172.17.0.5   minikube   <none>           <none>
web-6745ffd5c8-bpmfv   1/1     Running   0          28m   172.17.0.4   minikube   <none>           <none>
web-6745ffd5c8-zx75r   1/1     Running   0          28m   172.17.0.3   minikube   <none>           <none>
```
* Try connect to pod with curl (curl pod_ip_address). What happens?
* From my PC<br>
```bash
curl 172.17.0.3
<HTML><BODY><H1>301 Moved Permanently</H1></BODY></HTML>
```
Not conected
* From minikube (minikube ssh)
```bash
curl 172.17.0.3
web-6745ffd5c8-zx75r
```
Conected
* From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash)
```bash
curl 172.17.0.3
web-6745ffd5c8-zx75r
```
Conected
### Create service (ClusterIP)
The command that can be used to create a manifest template
```bash
kubectl expose deployment/web --type=ClusterIP --dry-run=client -o yaml > service_template.yaml
```
Apply manifest
```bash
kubectl apply -f service_template.yaml
```
Get service CLUSTER-IP
```bash
kubectl get svc
```
### My output
```bash
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   26h
web          ClusterIP   10.110.51.27   <none>        80/TCP    2m22s
```
* Try connect to service (curl service_ip_address). What happens?

* From my PC
```bash
curl 10.110.51.27
curl: (28) Failed to connect to 10.110.51.27 port 80: Connection timed out
```
Not conected
* From minikube (minikube ssh) (run the command several times)
```bash
curl 10.110.51.27
web-6745ffd5c8-zx75r
curl 10.110.51.27
web-6745ffd5c8-bpmfv
curl 10.110.51.27
web-6745ffd5c8-7lgf2
```
Conected
* From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash) (run the command several times)
```bash
curl 10.110.51.27
web-6745ffd5c8-bpmfv
curl 10.110.51.27
web-6745ffd5c8-zx75r
curl 10.110.51.27
web-6745ffd5c8-bpmfv
```
Conected
### NodePort
```bash
kubectl apply -f service-nodeport.yaml
kubectl get service
```
### My output
```bash
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        27h
web          ClusterIP   10.110.51.27    <none>        80/TCP         24m
web-np       NodePort    10.102.187.50   <none>        80:31996/TCP   87s
```
Note how port is specified for a NodePort service
### Checking the availability of the NodePort service type
```bash
minikube ip
192.168.49.2
curl 192.168.49.2:31996
web-6745ffd5c8-bpmfv
```
### Headless service
```bash
kubectl apply -f service-headless.yaml
```
### DNS
Compare the IP address of the DNS server in the pod 
```bash
cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```
and the DNS service of the Kubernetes cluster.
```bash
kubectl cluster-info
Kubernetes control plane is running at https://192.168.49.2:8443
CoreDNS is running at https://192.168.49.2:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```
Compare headless and clusterip.<br>
Inside the pod run nslookup to normal clusterip and headless. Compare the results.
You will need to create pod with dnsutils.
```bash
nslookup web
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web.default.svc.cluster.local
Address: 10.110.51.27

nslookup web-headless
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web-headless.default.svc.cluster.local
Address: 172.17.0.3
Name:   web-headless.default.svc.cluster.local
Address: 172.17.0.5
Name:   web-headless.default.svc.cluster.local
Address: 172.17.0.4
```
# Homework
* In Minikube in namespace kube-system, there are many different pods running. Your task is to figure out who creates them, and who makes sure they are running (restores them after deletion).<br>

kube-system - the namespace for objects created by the Kubernetes system
```bash
kubectl get all -n kube-system
NAME                                   READY   STATUS    RESTARTS      AGE
pod/coredns-64897985d-7zmx8            1/1     Running   2 (25h ago)   31h
pod/etcd-minikube                      1/1     Running   0             31h
pod/kube-apiserver-minikube            1/1     Running   0             31h
pod/kube-controller-manager-minikube   1/1     Running   0             31h
pod/kube-proxy-wcsrg                   1/1     Running   0             31h
pod/kube-scheduler-minikube            1/1     Running   0             31h
pod/storage-provisioner                1/1     Running   3 (25h ago)   31h

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   31h

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   31h

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           31h

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-64897985d   1         1         1       31h
```
They are controlled by higher level Kubernetes objects like Deployment, Replicaset, DaemonSet
```bash
kubectl get pods --namespace=kube-system -o custom-columns=NAME:.metadata.name,CONTROLLER:.metadata.ownerReferences[].kind,NAMESPACE:.metadata.namespace
NAME                               CONTROLLER   NAMESPACE
coredns-64897985d-7zmx8            ReplicaSet   kube-system
etcd-minikube                      Node         kube-system
kube-apiserver-minikube            Node         kube-system
kube-controller-manager-minikube   Node         kube-system
kube-proxy-wcsrg                   DaemonSet    kube-system
kube-scheduler-minikube            Node         kube-system
```
* Implement Canary deployment of an application via Ingress. Traffic to canary deployment should be redirected if you add "canary:always" in the header, otherwise it should go to regular deployment.
Set to redirect a percentage of traffic to canary deployment.

Add new data for the canary application to the existing configmap
```bash
vim nginx-configmap.yaml
apiVersion: v1
data:
  default.conf: |-
    server {
        listen 80 default_server;
        server_name _;
        default_type text/plain;

        location / {
            return 200 '$hostname\n';
        }
    }
  canary.conf: |-
    server {
        listen 80 default_server;
        server_name _;
        default_type text/plain;

        location / {
            return 200 'CANARY $hostname\n';
        }
    }
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: nginx-configmap
```
Modify the existing deployment to use the modified configmap
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
          - name: config-nginx
            mountPath: /etc/nginx/conf.d
      volumes:
        - name: config-nginx
          configMap:
            name: nginx-configmap
            items:
            - key: default.conf
              path: default.conf
```
Apply regular deployment
```bash
kubectl apply -f deployment.yaml
deployment.apps/web created
```
Apply regular (existing) service
```bash
kubectl apply -f service_template.yaml
service/web created
```
Create canary deployment
```bash
vim deployment-canary.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web-canary
  name: web-canary
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-canary
  template:
    metadata:
      labels:
        app: web-canary
    spec:
      containers:
      - image: nginx:latest
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
          - name: config-nginx
            mountPath: /etc/nginx/conf.d
      volumes:
        - name: config-nginx
          configMap:
            name: nginx-configmap
            items:
            - key: canary.conf
              path: canary.conf
```
Apply canary deployment
```bash
kubectl apply -f deployment-canary.yaml
deployment.apps/web-canary created
```
Create canary service
```bash
kubectl expose deployment/web-canary --type=ClusterIP --dry-run=client -o yaml > service_canary.yaml
kubectl apply -f service_canary.yaml
service/web-canary created
  ```
Apply regular (existing) ingress
```bash
kubectl apply -f ingress.yaml
ingress.networking.k8s.io/ingress-web created
```
Create canary ingress
```bash
vim ingress-canary.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-canary
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "canary"
    nginx.ingress.kubernetes.io/canary-weight: "20"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
             name: web-canary
             port:
                number: 80
```
Apply canary ingress
```bash
kubectl apply -f ingress-canary.yaml
ingress.networking.k8s.io/ingress-canary created
```
See all
```bash
kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/web-canary-7dfd8d9d7-b5ds6   1/1     Running   0          24h
pod/web-canary-7dfd8d9d7-l8t96   1/1     Running   0          24h
pod/web-canary-7dfd8d9d7-pvh2j   1/1     Running   0          24h
pod/web-fd5549696-cxwk4          1/1     Running   0          24h
pod/web-fd5549696-m7m5c          1/1     Running   0          24h
pod/web-fd5549696-zbd64          1/1     Running   0          24h

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        3d7h
service/web            ClusterIP   10.97.10.170    <none>        80/TCP         8m36s
service/web-canary     ClusterIP   10.100.107.43   <none>        80/TCP         11m
service/web-headless   ClusterIP   None            <none>        80/TCP         2d4h
service/web-np         NodePort    10.102.187.50   <none>        80:31996/TCP   2d4h

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web          3/3     3            3           2d5h
deployment.apps/web-canary   3/3     3            3           24h

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/web-6745ffd5c8         0         0         0       2d5h
replicaset.apps/web-canary-7dfd8d9d7   3         3         3       24h
replicaset.apps/web-f65dfb8d6          0         0         0       24h
replicaset.apps/web-fd5549696          3         3         3       24h
```
Try to connet 10 times to cluster ip via ingress
```bash
for i in $(seq 1 10); do curl -s $(minikube ip) ; sleep 0.1; done
web-fd5549696-m7m5c
web-fd5549696-zbd64
web-fd5549696-m7m5c
web-fd5549696-cxwk4
web-fd5549696-zbd64
web-fd5549696-m7m5c
CANARY web-canary-7dfd8d9d7-pvh2j
web-fd5549696-cxwk4
web-fd5549696-zbd64
CANARY web-canary-7dfd8d9d7-b5ds6
```
Add header "canary:always" in request
```bash
for i in $(seq 1 10); do curl -s -H "canary:always" $(minikube ip) ; sleep 0.1; done
CANARY web-canary-7dfd8d9d7-pvh2j
CANARY web-canary-7dfd8d9d7-l8t96
CANARY web-canary-7dfd8d9d7-b5ds6
CANARY web-canary-7dfd8d9d7-pvh2j
CANARY web-canary-7dfd8d9d7-pvh2j
CANARY web-canary-7dfd8d9d7-l8t96
CANARY web-canary-7dfd8d9d7-l8t96
CANARY web-canary-7dfd8d9d7-l8t96
CANARY web-canary-7dfd8d9d7-b5ds6
CANARY web-canary-7dfd8d9d7-pvh2j
```