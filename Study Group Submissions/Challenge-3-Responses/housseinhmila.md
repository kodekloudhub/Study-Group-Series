**Name**: Houssein Hmila

**Slack User ID**: U054SML63K5

**Email**: housseinhmila@gmail.com   

---

## Solution

### Solution Details

#### Step 1 (ConfigMap Creation):

Created a directory called deployment.
Inside the deployment directory, you created a file named default.conf and pasted the Nginx configuration into it.
Changed the permissions of default.conf to make it readable.
Created a ConfigMap named nginx-conf from the default.conf file.
#### Step 2, 3, and 4 (Deployment Creation, Volume Mount, and Service Exposure):

Created a YAML file named nginx-proxy-deployment.yaml containing the Deployment resource configuration with two containers (simple-webapp and nginx-proxy) and volume mounts for the Nginx configuration.
Applied the nginx-proxy-deployment.yaml using kubectl apply to create the deployment.
Exposed the deployment as a Kubernetes Service on port 80 using kubectl expose

### Code Snippet

```
mkdir deployment
cd deployment
vim default.conf
```
##### paste this config
```
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
##### run these commands:
```
chmod 666 default.conf 
kubectl create configmap nginx-conf --from-file=./default.conf
vim nginx-proxy-deployment.yaml
```
##### paste this code
```apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy-deployment
  labels:
    app: nginx
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
        image: kodekloud/simple-webapp
        ports:
        - containerPort: 8080
      - name: nginx-proxy
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
            - name: config-volume
              mountPath: /etc/nginx/conf.d/
      volumes:
        - name: config-volume
          configMap:
            name: nginx-conf
```
##### run this command:
```
kubectl apply -f nginx-proxy-deployment.yaml
kubectl expose deployment nginx-proxy-deployment --port=80 --target-port=80
```