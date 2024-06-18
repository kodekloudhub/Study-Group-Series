**Name**: Mohan Vaddella

**Email**: mohanvaddella@gmail.com

---

# Weekly Challenge: Daemonset Configuration in Kubernetes

## Proposed Solution

First, let’s create the pods `service-red` and `service-blue`:

```yaml
# service-red.yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-red
spec:
  containers:
  - name: service-red
    image: kodekloud/simple-webapp
    ports:
    - containerPort: 8080
    env:
    - name: APP_COLOR
      value: "red"
```

```yaml
# service-blue.yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-blue
spec:
  containers:
  - name: service-blue
    image: kodekloud/simple-webapp
    ports:
    - containerPort: 8080
    env:
    - name: APP_COLOR
      value: "blue"
```

Apply these configurations with `kubectl apply -f service-red.yaml` and `kubectl apply -f service-blue.yaml`.

Next, let’s create the ConfigMap `nginx-conf` with the provided Nginx configuration:

```yaml
kubectl create configmap nginx-conf --from-literal=default.conf='
server {
    listen 80;
    server_name localhost;

    location /service-red {
        proxy_pass http://service-red;
    }

    location /service-blue {
        proxy_pass http://service-blue;
    }
}'
```

Now, let’s define the DaemonSet `nginx-reverse-proxy-daemonset`:

```yaml
# nginx-reverse-proxy-daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-reverse-proxy-daemonset
spec:
  selector:
    matchLabels:
      name: nginx-reverse-proxy
  template:
    metadata:
      labels:
        name: nginx-reverse-proxy
    spec:
      containers:
      - name: nginx-reverse-proxy
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: conf
          mountPath: /etc/nginx/conf.d/default.conf
      volumes:
      - name: conf
        configMap:
          name: nginx-conf
```

Apply this configuration with `kubectl apply -f nginx-reverse-proxy-daemonset.yaml`.

Finally, you can validate the setup by accessing the service. You can use `kubectl port-forward` or `kubectl proxy` to access the service from your local machine. For example:

```yaml
kubectl port-forward daemonset/nginx-reverse-proxy-daemonset 8080:80
```

Then, you can access the service at `http://localhost:8080/service-red` and `http://localhost:8080/service-blue`.



