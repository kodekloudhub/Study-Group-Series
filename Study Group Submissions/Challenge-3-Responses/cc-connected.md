**Name**: cc-​connected

**Slack User ID**: U053SNUCHAM

**Email**: office@cc-connected.com

---

## Solution

### Solution Details

Create ConfigMap with desired nginx configuration specified in default.conf file.\
Create Deployment contained of 2 containers and a volume mapped to ConfigMap. In containers specification: \
	1) simple-webapp\
	2) nginx-proxy - with volumeMount containing volume's name and mountPath directory location of default.conf file\
Create a service to make the deployment accessible within cluster.\
Create a temporary pod to test response from service.

### Code Snippet

```yaml
vi nginx-conf.yaml
```

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

```yaml
kubectl apply -f nginx-conf.yaml
```

```yaml
kubectl get configmap
```

```yaml
vi nginx-proxy-deployment.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: simple-webapp
        image: kodekloud/simple-webapp:latest
        ports:
        - containerPort: 8080
      - name: nginx-proxy
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-vol-config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-vol-config
        configMap:
          name: nginx-conf
```

```yaml
kubectl apply -f nginx-proxy-deployment.yaml
```

```yaml
kubectl get deploy -o=wide -w
```

```yaml
kubectl get pods -o=wide -w
```

```yaml
kubectl describe pod nginx-proxy-deployment-867bbd6b9b-szlqs
```

```yaml
kubectl expose deployment nginx-proxy-deployment --port=80 --target-port=80 --name=nginx-service
```

```yaml
kubectl get service -o=wide -w   #[Type:ClusterIP   Cluster-Ip:10.107.147.92   External-Ip:<none>        Port:80/TCP]
```

```yaml
kubectl run curltestpod --image=curlimages/curl -it --rm --restart=Never -- curl 10.107.147.92:80
```

```yaml
kubectl logs nginx-proxy-deployment-867bbd6b9b-szlqs
```
