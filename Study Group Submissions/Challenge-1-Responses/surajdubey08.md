
**Name**: Suraj Dhar Dubey

**Email**: surajdhar08.dubey@gmail.com

---

## Solution

### Solution Details

**Task 1: Deployment Configuration**

When tackling the task of setting up deployments for the development and production applications on the Kubernetes cluster, I took a methodical approach to ensure everything was in place for proper resource management.

Firstly, I wanted to create a deployment configuration for the development application. To achieve this, I put together a YAML file named `development-deployment.yaml`. Within this file, I outlined the Deployment resource specifications according to the requirements. I made sure to set the number of replicas to 1, added clear labels (`app: dev-app`) to distinguish the application, and incorporated affinity settings. The affinity part was particularly crucial; I utilized it to guarantee that the pods would be scheduled on nodes with the label `environment=development`.

Next, I moved on to the production application. Following the same approach, I crafted another YAML file called `production-deployment.yaml`. This file mirrored the previous one in terms of structure but was customized to fit the production environment's needs. I ensured the right number of replicas (3 in this case), labeled the application (`app: prod-app`), and configured affinity to make certain that pods were deployed to nodes marked with `environment=production`.

In the task, the `requiredDuringSchedulingIgnoredDuringExecution` affinity setting was used to ensure that the pods of the applications were scheduled on nodes with the desired environment label (`development` or `production`). This affinity setting enforces the placement of pods on nodes based on specified rules during scheduling and maintains that placement even if a node label changes during runtime execution.

Once the YAML files were ready, I used the `kubectl apply -f` command to put the deployment configurations into action. This step allowed the applications to be set up and the associated configurations to take effect within the Kubernetes cluster.

**Task 2: Log Inspection**

To accomplish this, I considered a straightforward yet effective approach. I opted for the `kubectl logs` command. By utilizing this command and specifying `-l app=prod-app`, I was able to specifically target the pods that belonged to the production application. This meant I could access and review the logs from these particular pods, ensuring everything was functioning as expected.

### Code Snippet
**Task 1: Deployment Configuration**

development-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dev-app
  template:
    metadata:
      labels:
        app: dev-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: environment
                    operator: In
                    values:
                      - development
```

production-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: prod-app
  template:
    metadata:
      labels:
        app: prod-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: environment
                    operator: In
                    values:
                      - production
```

Apply the Manifests:
```bash
kubectl apply -f development-deployment.yaml
kubectl apply -f production-deployment.yaml
```
**Task 2: Log Inspection**
```bash
kubectl logs -l app=prod-app
```