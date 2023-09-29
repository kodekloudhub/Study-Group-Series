**Name**: Jothilal Sottallu

**Email**: srjothilal2001@gmail.com


### SOLUTION


### TASK: Kubernetes Cluster Administration Task

### SOLUTION DETAILS

### 1. Create a ServiceAccount

* To create a service account, Ran the command -

```
└─(17:35:43)──> kbt create serviceaccount developer                                                                                                                                                                     
+kubectl
serviceaccount/developer created
```
* To check the service account 

```
kbt get serviceaccount developer                                                                                                                                                                     
+kubectl
NAME        SECRETS   AGE
developer   0         48s
```

### 2. Create an API token for the developer ServiceAccount.

* To create a API token for the Developer
  
```
kubectl create secret generic developer-api-token --from-literal=token=$(openssl rand -base64 32)

secret/developer-api-token created
```
* To check and decode the token 
  
    * We need to extract the token that Kubernetes automatically creates when you create a ServiceAccount
  
```
kubectl get secret developer-api-token -o jsonpath='{.data.token}' | base64 -d
PDpk0i8j3PSips/zoGITX0qulPCfdQwi4L0yOpQqR2E=%
```

### 3. Create a ClusterRole named read-deploy-pod-clusterrole with the following permissions:
Verbs: get, list, watch
Resources: pods, pods/log, deployments

* To create a clusterRole 

```
kubectl create clusterrole read-deploy-pod-clusterrole \
--verb=get,list,watch \
--resource=pods,pods/log,deployments

clusterrole.rbac.authorization.k8s.io/read-deploy-pod-clusterrole created
```
* Will verify this using kbt get command

```
kubectl get clusterrole read-deploy-pod-clusterrole                                                                                                                                                   
NAME                          CREATED AT
read-deploy-pod-clusterrole   2023-09-27T12:30:30Z
```

### 4. Create a ClusterRoleBinding


```
kubectl create clusterrolebinding developer-read-deploy-pod \                                                                                      
  --clusterrole=read-deploy-pod-clusterrole \
  --serviceaccount=default:developer

clusterrolebinding.rbac.authorization.k8s.io/developer-read-deploy-pod created
```

### 5. Generate a kubeconfig file for the developer.

    kubectl config set-cluster my-cluster \
    --kubeconfig=$HOME/path/to/developer-kubeconfig.yaml \
    --server=https://192.168.49.2:8443 \
    --insecure-skip-tls-verify=true

    Cluster "my-cluster" set.

    Before this I faced few issues related with APISERVER because I am using minikube and I need to find the apiserver to configure or set the cluster 
    minikube ip
    192.168.49.2 PORT for the minikube:8443

    kubectl config set-credentials developer \
    --kubeconfig=$HOME/path/to/developer-kubeconfig.yaml \
    --token=$(kubectl get secret developer-api-token -o jsonpath='{.data.token}' | base64 -d)
    User "developer" set.

     kubectl config set-context developer-context \
      --kubeconfig=$HOME/path/to/developer-kubeconfig.yaml \
      --cluster=my-cluster \
      --user=developer \
      --namespace=default
     Context "developer-context" created.

     kubectl config use-context developer-context \
      --kubeconfig=$HOME/path/to/developer-kubeconfig.yaml
     Switched to context "developer-context".    

* yaml
```
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://192.168.49.2:8443
  name: my-cluster
contexts:
- context:
    cluster: my-cluster
    namespace: default
    user: developer
  name: developer-context
current-context: developer-context
kind: Config
preferences: {}
users:
- name: developer
  user:
    token: PDpk0i8j3PSips/zoGITX0qulPCfdQwi4L0yOpQqR2E=
```