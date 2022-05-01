# Task 3
### [Read more about CSI](https://habr.com/ru/company/flant/blog/424211/)
### Create pv in kubernetes
```bash
kubectl apply -f pv.yaml
```
### Check our pv
```bash
kubectl get pv
```
### Sample output
```bash
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
minio-deployment-pv   5Gi        RWO            Retain           Available                                   5s
```
### Create pvc
```bash
kubectl apply -f pvc.yaml
```
### Check our output in pv 
```bash
kubectl get pv
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                            STORAGECLASS   REASON   AGE
minio-deployment-pv   5Gi        RWO            Retain           Bound    default/minio-deployment-claim                           94s
```
Output is change. PV get status bound.
### Check pvc
```bash
kubectl get pvc
NAME                     STATUS   VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-deployment-claim   Bound    minio-deployment-pv   5Gi        RWO                           79s
```
### Apply deployment minio
```bash
kubectl apply -f deployment.yaml
```
### Apply svc nodeport
```bash
kubectl apply -f minio-nodeport.yaml
```
Open minikup_ip:node_port in you browser
### Apply statefulset
```bash
kubectl apply -f statefulset.yaml
```
### Check pod and statefulset
```bash
kubectl get pod
kubectl get sts
```

### Homework
* We published minio "outside" using nodePort. Do the same but using ingress.

Enable Ingress controller
```bash
minikube addons enable ingress
```
Create ingress manifest
```bash
vim ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-minio
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
             name: minio-app
             port:
                number: 9001
```
Apply manifest 
```bash
kubectl apply -f ingress.yaml
```
Open ip_minikube in a browser

![pic1](https://github.com/haserge/kubernetes-homework/blob/main/task_3/console.JPG?raw=true)
* Publish minio via ingress so that minio by ip_minikube and nginx returning hostname (previous job) by path ip_minikube/web are available at the same time.

Apply manifests from the previous task
```bash
kubectl apply -f ../task_2/nginx-configmap.yaml
kubectl apply -f ../task_2/deployment.yaml
kubectl apply -f ../task_2/service_template.yaml
```
See pods
```bash
kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
minio-94fd47554-jfrdq   1/1     Running   0          16h
minio-state-0           1/1     Running   0          16h
web-fd5549696-b757c     1/1     Running   0          42m
web-fd5549696-gcpzq     1/1     Running   0          42m
web-fd5549696-jvp4n     1/1     Running   0          42m
```
See services
```bash
kubectl get services
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP          8d
minio-app     NodePort    10.110.95.105    <none>        9001:30008/TCP   16h
minio-state   ClusterIP   None             <none>        9000/TCP         16h
web           ClusterIP   10.111.228.205   <none>        80/TCP           41m
```
Edit existing ingress
```bash
vim ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-minio
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
             name: minio-app
             port:
                number: 9001
      - path: /web
        pathType: Prefix
        backend:
          service:
             name: web
             port:
                number: 80
```
Apply ingress
```bash
kubectl apply -f ingress.yaml
```
Open ip_minikube and ip_minikube/web in a browser

![pic2](https://github.com/haserge/kubernetes-homework/blob/main/task_3/minio_nginx.JPG?raw=true)

* Create deploy with emptyDir save data to mountPoint emptyDir, delete pods, check data.

Create deploy for Minio with storage emptyDir 
```bash
cp deployment.yaml minio_empty_dir.yaml
vim minio_empty_dir.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: minio-empty-dir
  labels:
    app: minio-empty-dir
spec:
  replicas: 1
  selector:
    matchLabels:
      app: minio-empty-dir
  template:
    metadata:
      labels:
        app: minio-empty-dir
    spec:
      containers:
        - name: minio
          image: minio/minio
          args: ["server", "/data", "--console-address", ":9001"]
          ports:
            - containerPort: 9001
          volumeMounts:
            - name: data
              mountPath: "data"
              readOnly: false
      volumes:
        - name: data
          emptyDir: {}
```
Apply deploy
```bash
kubectl apply -f minio_empty_dir.yaml
```
Create file in volume from pod
```bash
kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
minio-94fd47554-jfrdq              1/1     Running   0          20h
minio-empty-dir-67fdd78b95-x7f2c   1/1     Running   0          78s
minio-state-0                      1/1     Running   0          19h

kubectl exec -it minio-empty-dir-67fdd78b95-x7f2c bash
[root@minio-empty-dir-67fdd78b95-x7f2c /]# touch /data/test_datafile
```
Check volume data
```bash
[root@minio-empty-dir-67fdd78b95-x7f2c /]# ls -la /data
total 12
drwxrwxrwx 3 root root 4096 May  1 13:23 .
drwxr-xr-x 1 root root 4096 May  1 13:20 ..
drwxr-xr-x 6 root root 4096 May  1 13:20 .minio.sys
-rw-r--r-- 1 root root    0 May  1 13:23 test_datafile
```
Delete pod
```bash
kubectl delete po minio-empty-dir-67fdd78b95-x7f2c
pod "minio-empty-dir-67fdd78b95-x7f2c" deleted
```
Connect new pod
```bash
kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
minio-94fd47554-jfrdq              1/1     Running   0          20h
minio-empty-dir-67fdd78b95-phgkp   1/1     Running   0          36s
minio-state-0                      1/1     Running   0          20h

kubectl exec -it minio-empty-dir-67fdd78b95-phgkp bash
[root@minio-empty-dir-67fdd78b95-phgkp /]# ls -la /data
total 12
drwxrwxrwx 3 root root 4096 May  1 13:31 .
drwxr-xr-x 1 root root 4096 May  1 13:31 ..
drwxr-xr-x 6 root root 4096 May  1 13:31 .minio.sys
```
See that file "test_datafile" are gone.