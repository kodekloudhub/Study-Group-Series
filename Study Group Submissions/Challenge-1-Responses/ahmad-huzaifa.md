# Weekly Challenge: Deployment Configuration and Log Inspection in Kubernetes

## Task 1: Deployment Configuration

**Objective:** Configure deployments for development and production applications using the `nginx:latest` image, ensuring appropriate scheduling on labeled nodes.

**Development Application (`development-app.yaml`):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-app
  labels:
    app: dev-app
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
      - name: deployment-app
        image: nginx:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
```



**Production Application (`production-app.yaml`):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
  labels:
    app: prod-app
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
      - name: production-app
        image: nginx:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
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

```
huzafa [ ~ ]$ kubectl logs -l app=prod-app
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/21 10:43:33 [notice] 1#1: using the "epoll" event method
2023/09/21 10:43:33 [notice] 1#1: nginx/1.25.2
2023/09/21 10:43:33 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2023/09/21 10:43:33 [notice] 1#1: OS: Linux 5.15.0-1042-azure
2023/09/21 10:43:33 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/21 10:43:33 [notice] 1#1: start worker processes
2023/09/21 10:43:33 [notice] 1#1: start worker process 29
2023/09/21 10:43:33 [notice] 1#1: start worker process 30
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/21 10:43:32 [notice] 1#1: using the "epoll" event method
2023/09/21 10:43:32 [notice] 1#1: nginx/1.25.2
2023/09/21 10:43:32 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2023/09/21 10:43:32 [notice] 1#1: OS: Linux 5.15.0-1042-azure
2023/09/21 10:43:32 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/21 10:43:32 [notice] 1#1: start worker processes
2023/09/21 10:43:32 [notice] 1#1: start worker process 29
2023/09/21 10:43:32 [notice] 1#1: start worker process 30
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/21 10:43:31 [notice] 1#1: using the "epoll" event method
2023/09/21 10:43:31 [notice] 1#1: nginx/1.25.2
2023/09/21 10:43:31 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2023/09/21 10:43:31 [notice] 1#1: OS: Linux 5.15.0-1042-azure
2023/09/21 10:43:31 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/21 10:43:31 [notice] 1#1: start worker processes
2023/09/21 10:43:31 [notice] 1#1: start worker process 30
2023/09/21 10:43:31 [notice] 1#1: start worker process 31
```
```
kubectl logs -l app=dev-app
```

This command retrieves logs from all containers within the pods that match the label selector `app=dev-app`. It helps you monitor and troubleshoot the deployment environment effectively.

```
huzafa [ ~ ]$ kubectl logs -l app=dev-app
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/21 10:32:22 [notice] 1#1: using the "epoll" event method
2023/09/21 10:32:22 [notice] 1#1: nginx/1.25.2
2023/09/21 10:32:22 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2023/09/21 10:32:22 [notice] 1#1: OS: Linux 5.15.0-1042-azure
2023/09/21 10:32:22 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/21 10:32:22 [notice] 1#1: start worker processes
2023/09/21 10:32:22 [notice] 1#1: start worker process 28
2023/09/21 10:32:22 [notice] 1#1: start worker process 29
```

