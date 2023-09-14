**Name**: Kostiantyn Zaihraiev

**Slack User ID**: U04FZK4HL0L

**Email**: kostya.zaigraev@gmail.com

---

## Solution

### Solution Details

After reading task, I've decided to pregenerate yaml templates to easy configuration and management of all resources.

First of all I've created `nginx.conf` file in which I've filled in with provided Nginx configuration.

Then I've pregenerated `configmap.yaml` file using `kubectl` command:

```bash
kubectl create configmap nginx-conf --from-file=nginx.conf --dry-run=client -o yaml > configmap.yaml
```

After making some changes in the file I've applied it to the cluster:

```bash
kubectl apply -f configmap.yaml
```

Then I've pregenerated `deployment.yaml` file in which I've defined a deployment named `nginx-proxy-deployment` with two containers: `simple-webapp` and `nginx-proxy` using `kubectl` command:

```bash
kubectl create deployment nginx-proxy-deployment --image=kodekloud/simple-webapp --image=nginx --dry-run=client -o yaml > deployment.yaml
```

After making some changes (including mounting the `nginx-conf` ConfigMap to the path `/etc/nginx/conf.d/default.conf` in the `nginx-proxy` container) in the file I've applied it to the cluster:

```bash
kubectl apply -f deployment.yaml
```

Finally I've exposed the `nginx-proxy` deployment on port 80 (I've used `NodePort` type of service to expose deployment).

```bash
kubectl expose deployment nginx-proxy-deployment --port=80 --type=NodePort --dry-run=client -o yaml > service.yaml
```

After making some changes in the file I've applied it to the cluster:

```bash
kubectl apply -f service.yaml
```

### Code Snippet

#### configmap.yaml
```yaml
apiVersion: v1
data:
  nginx.conf: |-
    server {
      listen 80;
      server_name localhost;
      location / {
          proxy_set_header   X-Forwarded-For $remote_addr;
          proxy_set_header   Host $http_host;
          proxy_pass         "http://127.0.0.1:8080";
      }
    }
kind: ConfigMap
metadata:
  name: nginx-conf
```

#### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-proxy-deployment
  name: nginx-proxy-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-proxy-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-proxy-deployment
    spec:
      containers:
      - image: kodekloud/simple-webapp
        name: simple-webapp
        resources: {}
        ports:
        - containerPort: 8080
      - image: nginx
        name: nginx-proxy
        resources: {}
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: nginx.conf
      volumes:
      - name: nginx-conf
        configMap:
          name: nginx-conf
```

#### service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-proxy-deployment
  name: nginx-proxy-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-proxy-deployment
  type: NodePort # I've used NodePort type of service to expose deployment
```
