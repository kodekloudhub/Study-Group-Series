**Name**: Basil  
**Slack User ID**: U05QWDXNWLF  
**Email**: basil-is@superhero.true

---

### Solution:

#### Step 1: Create a ServiceAccount

```bash
k create sa developer -n default
```

#### Step 2: Generate an API Token

We'll generate the token and store it in env var for future use.

```bash
export TOKEN=$(k create token developer)
```

#### Step 3. Define a ClusterRole

```bash
k create clusterrole read-deploy-pod-clusterrole --verb=get,list,watch --resource=pods,pods/log,deployments
```

#### Step 4. Create a ClusterRoleBinding

```bash
k create clusterrolebinding developer-read-deploy-pod --clusterrole=read-deploy-pod-clusterrole --serviceaccount=default:developer
```

#### Step 5. Generate a kubeconfig file

First, we need to find out the existing configuration to use command `kubeadm kubeconfig`. Normally, it stored in `~/.kube/config`.

But let's consider that we are in a custom setup where it's not in the default folder.

First, we need to find the kubeconfig path in the kubectl parameters using `ps aux` and `grep` combination.

We store the value in env variable:

```bash
export CONFIG_PATH=$(ps aux | grep kubectl | grep -oP '(?<=--kubeconfig )[^ ]+')
```

Let's check the value:

```bash
echo $CONFIG_PATH
/root/.kube/config
```

In case it's empty, we anyway can get the configuration from ConfigMap from the cluster itself:

```
kubectl get cm kubeadm-config -n kube-system -o=jsonpath="{.data.ClusterConfiguration}" > ~/clean-config
export CONFIG_PATH=~/clean-config
```

**NOTE:** This is preferred way, since your local config file could potentially already contain other users with their tokens. Getting the config from ConfigMap gives you a _clean_ config file with only cluster info.

Now, we use `kubeadm` command to add a user

```bash
kubeadm kubeconfig user --client-name=basil --token=$TOKEN --config=$CONFIG_PATH > config-basil.yaml
```

Locally, we could test this out by running next commands:

```bash
k get pods -A --kubeconfig=config-basil.yaml
(success)
```

```bash
k get svc -A --kubeconfig=config-basil.yaml
(failed: no access to services)
```
