# Weekly Challenge: Deployment Configuration and Log Inspection in Kubernetes

## Task 1: Deployment Configuration

**Objective:** Configure deployments for development and production applications using the `nginx:latest` image, ensuring appropriate scheduling on labeled nodes.

**Development Application (`development-app.yaml`):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: development-app
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
        - name: app-container
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



**Production Application (`production-app.yaml`):**

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
        - name: app-container
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

To deploy these configurations, you can apply the manifests using the following commands:

```
kubectl apply -f development-app.yaml

kubectl apply -f production-app.yaml
```

## Task 2: Log Inspection

After successfully deploying the applications, you can inspect the logs of the three pods within the production environment using the following command:

```
kubectl logs -l app=prod-app
```

This command retrieves logs from all containers within the pods that match the label selector `app=prod-app`. It helps you monitor and troubleshoot the production environment effectively.

By following these steps, you ensure that the development and production applications are appropriately configured and that you can access their logs for maintenance and debugging purposes.
