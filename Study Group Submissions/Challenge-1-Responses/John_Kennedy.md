**Name**: John Kennedy

**Email**: johnkenn777@gmail.com

---

## Solution

# Deployment Configuration and Log Inspection in Kubernetes 

## Task 1: Deployment Configuration

### Step 1: Label Nodes
Label the nodes in the cluster to specify where your pods will be created.

kubectl label node node01 environment=development
kubectl label node node02 environment=production

**Verification:**
To verify that the nodes have been labeled correctly, you can use the following commands:

kubectl get nodes --show-labels

### Step 2: Create Development Deployment
1. Create a YAML file for the development deployment:

kubectl create deploy deployment-app --image=nginx:latest --replicas=1 --dry-run=client -o yaml > nginx-dev-deploy.yaml

**Verification:**
Ensure the YAML file (`nginx-dev-deploy.yaml`) has been created successfully using the following command:

cat nginx-dev-deploy.yaml

2. Edit the YAML file using your preferred text editor (e.g., vi):

vi nginx-dev-deploy.yaml

3. Change the labels from `app: deployment-app` to `app: dev-app`.

4. Add the node affinity configuration under `template: spec` section to ensure scheduling on development nodes.

5. Save the changes and exit the text editor.

6. Apply the deployment configuration:

kubectl apply -f nginx-dev-deploy.yaml

**Verification:**
Check the status of the deployment and pods using the following commands:

kubectl get deploy -o wide -w
kubectl get po -o wide -w

### Step 3: Create Production Deployment
1. Create a YAML file for the production deployment:

kubectl create deploy production-app --image=nginx:latest --replicas=3 --dry-run=client -o yaml > nginx-prod-deploy.yaml

**Verification:**
Ensure the YAML file (`nginx-prod-deploy.yaml`) has been created successfully using the following command:

cat nginx-prod-deploy.yaml

2. Edit the YAML file using your preferred text editor:

vi nginx-prod-deploy.yaml

3. Change the labels from `app: production-app` to `app: prod-app`.

4. Add the node affinity configuration under `template: spec` section to ensure scheduling on production nodes.

5. Save the changes and exit the text editor.

6. Apply the production deployment configuration:

kubectl apply -f nginx-prod-deploy.yaml

**Verification:**
Check the status of the production deployment and pods using the following commands:

kubectl get deploy -o wide -w
kubectl get po -o wide -w

## Task 2: Log Inspection

After successfully deploying the applications, you need to inspect the logs of the three pods within the production environment.

kubectl logs production-app-86d6755579-4p24v
kubectl logs production-app-86d6755579-c49hr
kubectl logs production-app-86d6755579-d5gxq

**Verification:**
Run the above `kubectl logs` commands for each pod and ensure that you receive the logs from each pod as expected.

Remember to replace the pod names (`production-app-86d6755579-4p24v`, etc.) with the actual names of the pods running in your cluster.

## Solution Details

To solve this challenge, we followed these steps:

1. Labeled the appropriate nodes with the corresponding environments using `kubectl label` commands.
2. Created YAML files for deployment configurations using `kubectl create deploy` with the `--dry-run=client -o yaml` flags.
3. Edited the YAML files to modify labels and add node affinity configurations for the targeted environments using a text editor.
4. Applied the deployment configurations using `kubectl apply -f`.
5. Monitored the status of deployments and pods using `kubectl get` commands.
6. Inspected the logs of production pods using `kubectl logs`.

These steps ensured that the applications were correctly deployed and that logs could be inspected as required.

## Code Snippet

Here's an example of the node affinity configuration added to the YAML files for both development and production deployments:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: environment
              operator: In
              values:
                - development  # or 'production' for the production deployment
```

This configuration ensured that pods were scheduled on nodes with the appropriate environment labels.

