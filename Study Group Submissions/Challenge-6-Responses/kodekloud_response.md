# nginx-ingress-setup-solution

## wear-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wear-deployment
  name: wear-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wear-deployment
  template:
    metadata:
      labels:
        app: wear-deployment
    spec:
      containers:
      - image: kodekloud/ecommerce:apparels
        name: wear-deployment
        ports:
        - containerPort: 8080
```
---
## wear-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: wear-deployment
  name: wear-deployment
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: wear-deployment
```
---
## video-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: video-deployment
  name: video-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: video-deployment
  template:
    metadata:
      labels:
        app: video-deployment
    spec:
      containers:
      - image: kodekloud/ecommerce:video
        name: video-deployment
        ports:
        - containerPort: 8080
```
---
## video-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: video-deployment
  name: video-deployment
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: video-deployment
```
---
## ecommerce-ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: cka.kodekloud.local
    http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-deployment
            port:
              number: 8080
      - path: /video
        pathType: Prefix
        backend:
          service:
            name: video-deployment
            port:
              number: 8080
```

### Add IP Node cka.kodekloud.local to file host (e.g., /etc/hosts) and test it.

### DNS Configuration (if needed):
1. Add cka.kodekloud.local to configmap/coredns.
2. Restart deployment/coredns.
3. Test the setup:
####   kubectl run curl --image curlimages/curl:7.65.3 --command -- curl http://cka.kodekloud.local/wear
