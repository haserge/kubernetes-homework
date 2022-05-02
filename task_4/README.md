# Homework
* Create users deploy_view and deploy_edit. Give the user deploy_view rights only to view deployments, pods. Give the user deploy_edit full rights to the objects deployments, pods.

Configure users authentication using x509 certificates.<br>
Create private keys for users
```bash
openssl genrsa -out deploy_view.key 2048
openssl genrsa -out deploy_edit.key 2048
```
Create a certificate signing requests
```bash
openssl req -new -key deploy_view.key -out deploy_view.csr -subj "/CN=deploy_view"
openssl req -new -key deploy_edit.key -out deploy_edit.csr -subj "/CN=deploy_edit"
```
Sign the CSRs in the Kubernetes CA. We have to use the CA certificate and the key, which are usually in /etc/kubernetes/pki. But since we use minikube, the certificates will be on the host machine in ~/.minikube
```bash
openssl x509 -req -in deploy_view.csr \
-CA ~/.minikube/ca.crt \
-CAkey ~/.minikube/ca.key \
-CAcreateserial \
-out deploy_view.crt -days 100
openssl x509 -req -in deploy_edit.csr \
-CA ~/.minikube/ca.crt \
-CAkey ~/.minikube/ca.key \
-CAcreateserial \
-out deploy_edit.crt -days 100
```
Create users in Kubernetes
```bash
kubectl config set-credentials deploy_view \
--client-certificate=deploy_view.crt \
--client-key=deploy_view.key
User "deploy_view" set.
kubectl config set-credentials deploy_edit \
--client-certificate=deploy_edit.crt \
--client-key=deploy_edit.key
User "deploy_edit" set.
```
Set context for users
```bash
kubectl config set-context deploy_view --cluster=minikube --user=deploy_view
Context "deploy_view" created.
kubectl config set-context deploy_edit --cluster=minikube --user=deploy_edit
Context "deploy_edit" created.
```
Create RBAC Role and RoleBinding for "deploy_view" user
```bash
vim deploy_view_role-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployments-pods-view
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: deployments-pods-view-binding
subjects:
- kind: User
  name: deploy_view
roleRef:
  kind: ClusterRole
  name: deployments-pods-view
  apiGroup: rbac.authorization.k8s.io
```
Apply RBAC Role and RoleBinding for "deploy_view" user
```bash
kubectl apply -f deploy_view_role-rolebinding.yaml
clusterrole.rbac.authorization.k8s.io/deployments-pods-view created
clusterrolebinding.rbac.authorization.k8s.io/deployments-pods-view-binding created
```
Check privileges for "deploy_view" user 
```bash
kubectl config use-context deploy_view
Switched to context "deploy_view".
kubectl auth can-i get deployments
yes
kubectl auth can-i delete deployments
no
kubectl auth can-i get pods
yes
kubectl auth can-i delete pods
no
kubectl auth can-i create services
no
```
Create RBAC Role and RoleBinding for "deploy_edit" user
```bash
cp deploy_view_role-rolebinding.yaml deploy_edit_role-rolebinding.yaml
vim deploy_edit_role-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployments-pods-edit
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["*"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: deployments-pods-edit-binding
subjects:
- kind: User
  name: deploy_edit
roleRef:
  kind: ClusterRole
  name: deployments-pods-edit
  apiGroup: rbac.authorization.k8s.io
```
Apply RBAC Role and RoleBinding for "deploy_edit" user
```bash
kubectl apply -f deploy_edit_role-rolebinding.yaml
clusterrole.rbac.authorization.k8s.io/deployments-pods-edit created
clusterrolebinding.rbac.authorization.k8s.io/deployments-pods-edit-binding created
```
Check privileges for "deploy_edit" user 
```bash
kubectl config use-context deploy_edit
Switched to context "deploy_edit".
kubectl auth can-i get deployments
yes
kubectl auth can-i delete deployments
yes
kubectl auth can-i get pods
yes
kubectl auth can-i delete pods
yes
kubectl auth can-i create pods
yes
kubectl auth can-i create services
no
```
* Create namespace prod. Create users prod_admin, prod_view. Give the user prod_admin admin rights on ns prod, give the user prod_view only view rights on namespace prod.

Create the namespace "prod"
```bash
kubectl create namespace prod
```
Configure users authentication using x509 certificates.<br>
Create private keys for users
```bash
openssl genrsa -out prod_admin.key 2048
openssl genrsa -out prod_view.key 2048
```
Create a certificate signing requests
```bash
openssl req -new -key prod_admin.key -out prod_admin.csr -subj "/CN=prod_admin"
openssl req -new -key prod_view.key -out prod_view.csr -subj "/CN=prod_view"
```
Sign the CSRs in the Kubernetes CA. We have to use the CA certificate and the key, which are usually in /etc/kubernetes/pki. But since we use minikube, the certificates will be on the host machine in ~/.minikube
```bash
openssl x509 -req -in prod_admin.csr \
-CA ~/.minikube/ca.crt \
-CAkey ~/.minikube/ca.key \
-CAcreateserial \
-out prod_admin.crt -days 100
openssl x509 -req -in prod_view.csr \
-CA ~/.minikube/ca.crt \
-CAkey ~/.minikube/ca.key \
-CAcreateserial \
-out prod_view.crt -days 100
```
Create users in Kubernetes
```bash
kubectl config set-credentials prod_admin \
--client-certificate=prod_admin.crt \
--client-key=prod_admin.key
User "prod_admin" set.
kubectl config set-credentials prod_view \
--client-certificate=prod_view.crt \
--client-key=prod_view.key
User "prod_view" set.
```
Set context for users
```bash
kubectl config set-context prod_admin --cluster=minikube --user=prod_admin
Context "prod_admin" created.
kubectl config set-context prod_view --cluster=minikube --user=prod_view
Context "prod_view" created.
```
Create RBAC Role and RoleBinding for "prod_admin" user
```bash
vim prod_admin_role-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 namespace: prod
 name: prod_admin-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prod_admin-binding
  namespace: prod
subjects:
- kind: User
  name: prod_admin
roleRef:
  kind: Role
  name: prod_admin-role
  apiGroup: rbac.authorization.k8s.io
```
Apply RBAC Role and RoleBinding for "prod_admin" user
```bash
kubectl apply -f prod_admin_role-rolebinding.yaml
role.rbac.authorization.k8s.io/prod_admin-role created
rolebinding.rbac.authorization.k8s.io/prod_admin-binding created
```
Check privileges for "prod_admin" user
```bash
kubectl auth can-i list pods --as prod_admin -n prod
yes
kubectl auth can-i list pods --as prod_admin -n default
no
kubectl auth can-i create pods --as prod_admin -n prod
yes
kubectl auth can-i delete pods --as prod_admin -n prod
yes
kubectl auth can-i create deploy --as prod_admin -n prod
yes
kubectl auth can-i create deploy --as prod_admin -n default
no
kubectl auth can-i delete deploy --as prod_admin -n prod
yes
kubectl auth can-i get deploy --as prod_admin -n prod
yes
kubectl auth can-i delete services --as prod_admin -n prod
yes
kubectl auth can-i delete rs --as prod_admin -n prod
yes
```
Create RBAC Role and RoleBinding for "prod_view" user
```bash
cp prod_admin_role-rolebinding.yaml prod_view_role-rolebinding.yaml
vim prod_view_role-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 namespace: prod
 name: prod_view-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prod_view-binding
  namespace: prod
subjects:
- kind: User
  name: prod_view
roleRef:
  kind: Role
  name: prod_view-role
  apiGroup: rbac.authorization.k8s.io
```
Apply RBAC Role and RoleBinding for "prod_view" user
```bash
kubectl apply -f prod_view_role-rolebinding.yaml
role.rbac.authorization.k8s.io/prod_view-role created
rolebinding.rbac.authorization.k8s.io/prod_view-binding created
```
Check privileges for "prod_view" user
```bash
kubectl auth can-i get pods --as prod_view -n prod
yes
kubectl auth can-i create pods --as prod_view -n prod
no
kubectl auth can-i get deploy --as prod_view -n prod
yes
kubectl auth can-i create deploy --as prod_view -n prod
no
kubectl auth can-i get services --as prod_view -n prod
yes
kubectl auth can-i delete services --as prod_view -n prod
no
kubectl auth can-i get pods --as prod_view -n default
no
kubectl auth can-i get deploy --as prod_view -n default
no
```
* Create a serviceAccount sa-namespace-admin. Grant full rights to namespace default. Create context, authorize using the created sa, check accesses.
```bash
kubectl create sa sa-namespace-admin
serviceaccount/sa-namespace-admin created
```
Create RBAC Role and RoleBinding for "sa-namespace-admin" user
```bash
cp prod_admin_role-rolebinding.yaml sa-namespace-admin-role-rolebinding.yaml
vim sa-namespace-admin-role-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 namespace: default
 name: sa-namespace-admin-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sa-namespace-admin-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: sa-namespace-admin
roleRef:
  kind: Role
  name: sa-namespace-admin-role
  apiGroup: rbac.authorization.k8s.io
```
Apply RBAC Role and RoleBinding for "sa-namespace-admin" user
```bash
kubectl apply -f sa-namespace-admin-role-rolebinding.yaml
role.rbac.authorization.k8s.io/sa-namespace-admin-role created
rolebinding.rbac.authorization.k8s.io/sa-namespace-admin-binding created
```
Create and set context
```bash
export SECRET_NAME=$(kubectl get sa sa-namespace-admin -o jsonpath='{.secrets[0].name}')
export SECRET=$(kubectl get secret $SECRET_NAME -o jsonpath='{.data.token}' | base64 --decode)
kubectl config set-credentials sa-namespace-admin --token=$SECRET
User "sa-namespace-admin" set.
kubectl config set-context sa-namespace-admin --cluster=minikube --user=sa-namespace-admin
Context "sa-namespace-admin" created.
```
Authorize using the created sa
```bash
kubectl config use-context sa-namespace-admin
Switched to context "sa-namespace-admin".
```
Check privileges
```bash
kubectl auth can-i create deploy
yes
kubectl auth can-i list pods
yes
kubectl auth can-i delete pods
yes
kubectl auth can-i delete services
yes
kubectl auth can-i create rs
yes
kubectl auth can-i create deploy -n prod
no
```
