**Name**: Jothilal Sottallu

**Email**: srjothilal2001@gmail.com


### SOLUTION

### TASK : Kubernetes Challenge: Ingress and Networking

### SOLUTION DETAILS

### 1. Create a wear-deployment:
Name: wear-deployment
Image: kodekloud/ecommerce:apparels
Port: 8080
Expose this deployment

First, I created the wear deployment file and then services 
#### YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wear-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wear
  template:
    metadata:
      labels:
        app: wear
    spec:
      containers:
      - name: wear-container
        image: kodekloud/ecommerce:apparels
        ports:
        - containerPort: 8080 
```
* Command to apply the Deployment 
```Kubectl apply -f wear-deployment.yaml```

### SERVICES
```
apiVersion: v1
kind: Service
metadata:
  name: wear-service
spec:
  selector:
    app: wear
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080

```
* Command to apply the Services
```Kubectl apply -f wear-services.yaml```


### 2. Create a video-deployment:

#### YAML
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: video-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: video
  template:
    metadata:
      labels:
        app: video
    spec:
      containers:
      - name: video-container
        image: kodekloud/ecommerce:apparels
        ports:
        - containerPort: 8080 
```
* Command to apply the Deployment 
```Kubectl apply -f video-deployment.yaml```

### SERVICES
```
apiVersion: v1
kind: Service
metadata:
  name: video-service
spec:
  selector:
    app: video
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080

```
* Command to apply the Services
```Kubectl apply -f video-services.yaml```

#### 3.Create an Ingress:
Name: ecommerce-ingress
Annotations: nginx.ingress.kubernetes.io/rewrite-target: /
Host: cka.kodekloud.local
Paths:
/wear routes to service wear
/video routes to service video

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: cka.kodekloud.local
    http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port:
              number: 8080
      - path: /video
        pathType: Prefix
        backend:
          service:
            name: video-service
            port:
              number: 8080

```
Apply the Ingress configuration using kubectl apply -f ecommerce-ingress.yaml

### 4.Point Virtual Domain

```
To point the virtual domain cka.kodekloud.local to cluster, need to modify local machine's DNS settings. Add an entry in machine's /etc/hosts

192.168.99.100 cka.kodekloud.local
```

### 5. Verify

1. Testing from Local Machine:

Open a web browser and access cka.kodekloud.local/wear and cka.kodekloud.local/video. The traffic should be routed to the respective services.

2. Testing from Cluster Pods:

To ensure that pods within the cluster can access cka.kodekloud.local, may need to ensure DNS resolution within the cluster. Kubernetes typically sets up DNS resolution automatically for pods.

```
kubectl exec -it wear-d6d98d -- curl http://cka.kodekloud.local/wear

```