# Homework 1
* Create a deployment nginx. Set up two replicas. Remove one of the pods, see what happens.
### Create a deployment nginx (file nginx_deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.12
        ports:
        - containerPort: 80
```
### Apply deployment and see deployment status
![pic1](https://github.com/haserge/kubernetes-homework/blob/main/task_1/1.JPG?raw=true)
### Get pods status
![pic2](https://github.com/haserge/kubernetes-homework/blob/main//task_1/2.JPG?raw=true)
### Remove one of the pods and see that another pod is running
![pic3](https://github.com/haserge/kubernetes-_homework/blob/main/task_1/3.JPG?raw=true)

