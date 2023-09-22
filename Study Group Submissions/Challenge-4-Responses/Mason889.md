**Name**: Kostiantyn Zaihraiev

**Slack User ID**: U04FZK4HL0L

**Email**: kostya.zaigraev@gmail.com

---

## Solution

### Solution Details

After reading task, I've decided to pregenerate yaml templates to easy configuration and management of all resources.

First of all I've pregenerated `serviceaccount.yaml` file using `kubectl` command:

```bash
kubectl create serviceaccount developer --dry-run=client -o yaml > serviceaccount.yaml
```

After making some changes in the file I've applied it to the cluster:

```bash
kubectl apply -f serviceaccount.yaml
```

Then I've pregenerated `secret.yaml` file using `kubectl` command:

```bash
kubectl create secret generic developer --dry-run=client -o yaml > secret.yaml
```

After making some changes in the file to use service account token I've applied it to the cluster:

```bash
kubectl apply -f secret.yaml
```

Then I've pregenerated `clusterrole.yaml` file using `kubectl` command:

```bash
kubectl create clusterrole read-deploy-pod-clusterrole --verb=get,list,watch --resource=pods,pods/log,deployments --dry-run=client -o yaml > clusterrole.yaml
```

After making some changes in the file I've applied it to the cluster:

```bash
kubectl apply -f clusterrole.yaml
```

Then I've pregenerated `clusterrolebinding.yaml` file using `kubectl` command:

```bash
kubectl create clusterrolebinding developer-read-deploy-pod --serviceaccount=default:developer --clusterrole=read-deploy-pod-clusterrole --dry-run=client -o yaml > clusterrolebinding.yaml
```

After making some changes in the file I've applied it to the cluster:

```bash
kubectl apply -f clusterrolebinding.yaml
```

#### Kubeconfig file creation

First of all I've saved service account token to the environment variable called `TOKEN` for easy usage:

```bash
export TOKEN=$(kubectl get secret $(kubectl get serviceaccount developer -o jsonpath='{.metadata.name}') -o jsonpath='{.data.token}' | base64 --decode)
```

Then I've added service account to the kubeconfig file:

```bash
kubectl config set-credentials developer --token=$TOKEN
```

Then I've changed current context to use service account:

```bash
kubectl config set-context --current --user=developer
```

Then I've saved kubeconfig file to the `developer.kubeconfig` file:

```bash
kubectl config view --minify --flatten > developer.kubeconfig
```

Then I've checked that I can get/list pods:

```bash
kubectl get pods -n kube-system
```

Result:
```bash
NAME                               READY   STATUS    RESTARTS        AGE
coredns-5d78c9869d-4b5qz           1/1     Running   0               5h54m
etcd-minikube                      1/1     Running   0               5h54m
kube-apiserver-minikube            1/1     Running   0               5h54m
kube-controller-manager-minikube   1/1     Running   0               5h54m
kube-proxy-rcbb7                   1/1     Running   0               5h54m
kube-scheduler-minikube            1/1     Running   0               5h54m
storage-provisioner                1/1     Running   1 (5h53m ago)   5h54
```

### Code Snippet

#### serviceaccount.yaml
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer
  namespace: default

```

#### secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: developer
  annotations:
    kubernetes.io/service-account.name: developer
type: kubernetes.io/service-account-token

```

#### clusterrole.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: read-deploy-pod-clusterrole
  namespace: default
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - pods/log
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
  - get
  - list
  - watch
```

#### clusterrolebinding.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: developer-read-deploy-pod
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: read-deploy-pod-clusterrole
subjects:
- kind: ServiceAccount
  name: developer
  namespace: default
```
