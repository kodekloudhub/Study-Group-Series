**Name**: Houssein Hmila

**Slack User ID**: U054SML63K5

**Email**: housseinhmila@gmail.com   

---

## Solution

### Solution Details

#### Create a ServiceAccount: 
We create a service account named "developer" in the default namespace. 

### Code Snippet
```
kubectl create serviceaccount developer
```
#### Generate an API Token: 
We create an API token for the "developer" service account. This token is used by the "developer" to authenticate when making API requests to the Kubernetes cluster.

### Code Snippet
```yaml
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: developer-secret
  annotations:
    kubernetes.io/service-account.name: developer
type: kubernetes.io/service-account-token
EOF
```
Run these commands to extract and save the token:
```
SECRET_NAME=$(kubectl get serviceaccount developer -o jsonpath='{.secrets[0].name}')
API_TOKEN=$(kubectl get secret $SECRET_NAME -o jsonpath='{.items[].data.token}' | base64 --decode)
```
#### Define a ClusterRole: 
We create a ClusterRole named "read-deploy-pod-clusterrole" with specific permissions. This ClusterRole defines what actions the developer is allowed to perform within the cluster. In this case, the permissions are limited to get, list, and watch actions on resources like pods, pod logs, and deployments.

### Code Snippet
```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     # "namespace" omitted since ClusterRoles are not namespaced
     name: read-deploy-pod-clusterrole
   rules:
   - apiGroups: [""]
     #
     # at the HTTP level, the name of the resource for accessing Secret
    # objects is "secrets"
     resources: ["pods","pods/log"]
     verbs: ["get", "watch", "list"]
   - apiGroups: ["apps"]
     resources: ["deployments"]
     verbs: ["get", "watch", "list"]
```
#### Create a ClusterRoleBinding: 
We create a ClusterRoleBinding named "developer-read-deploy-pod" that binds the "developer" ServiceAccount to the "read-deploy-pod-clusterrole" ClusterRole. 

### Code Snippet
```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   # This cluster role binding allows anyone in the "manager" group to re    ad secrets in any namespace.
   kind: ClusterRoleBinding
   metadata:
     name: developer-read-deploy-pod
   subjects:
   - kind: ServiceAccount
     namespace: default
     name: developer # Name is case sensitive
  roleRef:
    kind: ClusterRole
    name: read-deploy-pod-clusterrole
    apiGroup: rbac.authorization.k8s.io
```
#### Generate a kubeconfig File:
Finally, We generate a kubeconfig file for the "developer." This kubeconfig file contains the necessary credentials (in this case, the API token) and context settings to access the cluster. 

### Code Snippet
```
kubectl config set-credentials developer --token=$API_TOKEN
kubectl config set-context developer-context --user=developer --cluster=<your_cluster_name> 
kubectl config use-context developer-context
```
##### you can test this by running ```kubectl get pod``` and ```kubectl get secret``` and you will see the difference.

