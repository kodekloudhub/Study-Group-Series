**Name**: Basil  
**Slack User ID**: U05QWDXNWLF  
**Email**: basil-is@superhero.true

---

### Solution:

#### Step 1: Create a ConfigMap with the Nginx Configuration

The following manifest creates a `ConfigMap` named `nginx-conf` that contains the Nginx configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  default.conf: |
    server {
      listen 80;
      server_name localhost;
      location / {
          proxy_set_header   X-Forwarded-For $remote_addr;
          proxy_set_header   Host $http_host;
          proxy_pass         "http://127.0.0.1:8080";
      }
    }
```

#### Step 2: Define the Deployment

The next step is to create a deployment named `nginx-proxy-deployment` that consists of the two containers and mounts the earlier created `ConfigMap`:

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
  template:
    metadata:
      labels:
        app: nginx-proxy-deployment
    spec:
      containers:
        - image: kodekloud/simple-webapp
          name: simple-webapp
          ports:
            - containerPort: 8080
        - image: nginx
          name: nginx-proxy
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/conf.d
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-conf
```

#### Step 3: Expose the service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-proxy-service
spec:
  selector:
    app: nginx-proxy-deployment
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

#### Step 4: Validate the Deployment

1. Get the service IP:

```bash
kubectl get svc nginx-proxy-service
```

2. Test the setup:

Using `curl` or any HTTP client, send a request to the IP address retrieved:

```bash
curl 10.42.0.15
```

As expected the output is an HTML response from the `simple-webapp`:

```html
<!DOCTYPE html>
<title>Hello from Flask</title>
<body style="background: #2980b9;"></body>
<div
  style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;"
>
  <h1>Hello from nginx-proxy-deployment-56bb6db74f-2px7v!</h1>
</div>
```
