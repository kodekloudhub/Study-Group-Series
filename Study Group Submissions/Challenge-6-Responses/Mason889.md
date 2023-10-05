**Name**: Kostiantyn Zaihraiev

**Slack User ID**: U04FZK4HL0L

**Email**: kostya.zaigraev@gmail.com

---

## Solution

### Pre-requisites

Run minikube cluster with ingress addon enabled:
```bash
minikube addons enable ingress
```

### Solution Details

After reading task, I've decided to create yaml templates that will be used to easy configuration and management of all resources.

```bash
kubectl create deploy wear-deployment --image=kodekloud/ecommerce:apparels --dry-run=client -o yaml > wear-deployment.yaml
kubectl create deploy video-deployment --image=kodekloud/ecommerce:video --dry-run=client -o yaml > video-deployment.yaml
kubectl create service clusterip wear --tcp=80:8080 --dry-run=client -o yaml > wear-service.yaml
kubectl create service clusterip video --tcp=80:8080 --dry-run=client -o yaml > video-service.yaml
kubectl create ingress ecommerce-ingress --rule="cka.kodekloud.local/wear=wear:80" --rule="cka.kodekloud.local/video=video:80" --dry-run=client -o yaml > ingress.yaml
```

Making some changes in the files, I've created the following resources:
```bash
kubectl apply -f wear-deployment.yaml
kubectl apply -f video-deployment.yaml
kubectl apply -f wear-service.yaml
kubectl apply -f video-service.yaml
kubectl apply -f ingress.yaml
```

After that, I've added the following line to the `/etc/hosts` file:
```bash
sudo vi /etc/hosts
```

Inside the hosts file I've added the following line:
```
minikube-ip cka.kodekloud.local
```

After that I've tested the ingress:
```bash
curl cka.kodekloud.local/wear
---
HTTP/1.1 200 OK
Date: Thu, 05 Oct 2023 10:43:32 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 296
Connection: keep-alive



curl cka.kodekloud.local/video
---
HTTP/1.1 200 OK
Date: Thu, 05 Oct 2023 10:43:39 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 293
Connection: keep-alive

```

### Code Snippet

#### wear-deployment.yaml
```yaml
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
          resources: {}
          ports:
            - containerPort: 8080

```

#### video-deployment.yaml
```yaml
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
          image: kodekloud/ecommerce:video
          resources: {}
          ports:
            - containerPort: 8080
```

#### wear-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: wear
spec:
  selector:
    app: wear
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

#### video-service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: video
spec:
  selector:
    app: video
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

#### ingress.yaml
```yaml
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
                name: wear
                port:
                  number: 80
          - path: /video
            pathType: Prefix
            backend:
              service:
                name: video
                port:
                  number: 80
```