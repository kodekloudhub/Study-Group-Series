Name: Emily Achieng
Email: emillyachieng@gmail.com

# Task
You are tasked with setting up a daemonset named nginx-reverse-proxy-daemonset as reverse proxy and other 2 pods service-red and service-blue. Then you need to config the nginx-reverse-proxy-daemonset to forward the path /service-red to the service-red and likewise for service-blue

# Solution
## Step 1: Create the service-red and service-blue pods
Here, we're creating two pods named service-red and service-blue. These pods are like separate rooms where our web applications will live.

1. create the service-red pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-red
  labels:
    app: service-red
spec:
  containers:
  - name: service-red
    image: kodekloud/simple-webapp
    ports:
    - containerPort: 8080
    env:
    - name: APP_COLOR
      value: red
    restartPolicy: Always
```

2. create the service-blue pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-blue
  labels:
    app: service-blue
spec:
  containers:
  - name: service-blue
    image: kodekloud/simple-webapp
    ports:
    - containerPort: 8080
    env:
    - name: APP_COLOR
      value: blue
    restartPolicy: Always
```

## Step 2: Create services for service-red and service-blue
Services in Kubernetes are like traffic directors. They help route incoming requests to the right place. Here, we create two services: one for service-red and one for service-blue. These services ensure that requests find their way to the correct pods, even if those pods change or move around.

3. create service-red service
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    service: service-red
  name: service-red
spec:
  selector:
    app: service-red
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

4. create service-blue service
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    service: service-blue
  name: service-blue
spec:
  selector:
    app: service-blue
  type: ClusterIP
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

## Step 3: Create configmap for nginx configuration
ConfigMaps are like cheat sheets for applications. They hold configuration settings separately from the application code. We're creating one named nginx-conf to hold instructions for our Nginx server. Inside, we specify that requests to certain paths should go to specific places, like /service-red going to the service-red pod.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen 80;
        server_name localhost;

        location /service-red {
            proxy_pass http://service-red;
        }

        location /service-blue {
            proxy_pass http://service-blue;
        }
    }
```

## Step 4: Create daemonset for nginx reverse proxy
A DaemonSet ensures that a specific pod runs on every node in the Kubernetes cluster. Here, we're using it to deploy Nginx, which will act as a reverse proxy. Nginx listens for incoming requests and sends them to the right place based on the rules we set up in the ConfigMap.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-reverse-proxy
  labels:
    app: nginx-reverse-proxy
spec:
  selector:
    matchLabels:
      app: nginx-reverse-proxy
  template:
    metadata:
      labels:
        app: nginx-reverse-proxy
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-conf
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: default.conf
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config

```

## Step 5: Portforward
Finally, we want to make sure everything works as expected. We use port forwarding to temporarily expose our Nginx server to our local machine. Then, we visit specific URLs, like /service-red and /service-blue, to see if our requests are being correctly routed to the corresponding pods.

In simpler terms, we're essentially setting up a system that ensures requests to different paths end up at the right destinations, all managed and orchestrated by Kubernetes.

```bash
kubectl port-forward daemonset/nginx-reverse-proxy-daemonset 8080:80
```
They can now be accessed locally at:
```bash
http://localhost:8080/service-red
```

```bash
http://localhost:8080/service-blue
```

Another way could be executing inside the pod:
```bash
kubectl exec -it service-red -- cat /etc/nginx/conf.d/default.conf
```
```bash
kubectl exec -it service-blue -- cat /etc/nginx/conf.d/default.conf
```