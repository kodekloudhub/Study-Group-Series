**Name**: Jahswill Ovedhe

**Email**: Jahswilling@gmail.com

---

## Solution
create deployments with labels for dev and prod (matchLabels)
### Solution Details

1. I generated 2 files ```deployment-dev-app.yaml``` and ```deployment-prod-app.yaml``` using the command ```kubectl create deployment dev-app --image=nginx:latest --dry-run=client -o yaml > deployment-dev-app.yaml``` and ```kubectl create deployment prod-app --image=nginx:latest --dry-run=client -o yaml > deployment-prod-app.yaml``` respectively
2. I looked up the proper usage of affinity on the documentation and implemented it on both.

## Task 1: Deployment Configuration
### Code Snippet

```
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
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: environment
                    operator: In
                    values:
                      - development
      containers:
        - name: nginx
          image: nginx:latest

---

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
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: environment
                    operator: In
                    values:
                      - production
      containers:
        - name: nginx
          image: nginx:latest

```

## Task 2: Log Inspection
To inspect logs for a specific pod, I used ```kubectl logs <pod-name>```