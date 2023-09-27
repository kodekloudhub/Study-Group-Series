# Deployment Configuration in Kubernetes

You are tasked with setting up a deployment named `nginx-proxy-deployment` that includes two containers: `simple-webapp` and `nginx-proxy`.

## Solution

To accomplish this task, follow these steps:

1. **Create a ConfigMap for Nginx Configuration (`nginx-conf`):**

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

2. **Define a Deployment (nginx-proxy-deployment) with Two Containers (simple-webapp and nginx-proxy):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy-deployment
  labels:
    app: nginx-proxy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-proxy
  template:
    metadata:
      labels:
        app: nginx-proxy
    spec:
      volumes:
        - name: nginx-conf
          configMap:
            name: nginx-conf
            items:
              - key: default.conf
                path: default.conf
      containers:
        - name: nginx-proxy
          image: nginx
          ports:
            - containerPort: 80
              protocol: TCP
          volumeMounts:
            - name: nginx-conf
              readOnly: true
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: default.conf
        - name: simple-webapp
          image: kodekloud/simple-webapp
          ports:
            - containerPort: 8080
              protocol: TCP

```


3. **Create a Service (nginx-proxy-deployment) to Expose the Deployment:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-proxy-deployment
  labels:
    app: nginx-proxy
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  selector:
    app: nginx-proxy
  type: ClusterIP
```

4. **Validate the Setup:**

Access the service to ensure it's correctly configured. You should observe content from the simple-webapp response.

This solution sets up a Kubernetes Deployment named nginx-proxy-deployment with two containers, simple-webapp and nginx-proxy, and ensures that the Nginx configuration is correctly applied using a ConfigMap.