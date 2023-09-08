# Deployment Configuration in Kubernetes

You are tasked with setting up a deployment named `nginx-proxy-deployment` that includes two containers: `simple-webapp` and `nginx-proxy`.

1. **Container Name `simple-webapp`:**
   - Image: `kodekloud/simple-webapp`
   - Port: 8080

2. **Container Name `nginx-proxy`:**
   - Image: `nginx`
   - Port: 80
   - Nginx Configuration:
   ```nginx
   server {
     listen 80;
     server_name localhost;
     location / {
         proxy_set_header   X-Forwarded-For $remote_addr;
         proxy_set_header   Host $http_host;
         proxy_pass         "http://127.0.0.1:8080";
     }
   }


To accomplish this task, follow these steps:

* Create a ConfigMap named nginx-conf with the provided Nginx configuration.

* Define a deployment named nginx-proxy-deployment with two containers: simple-webapp and nginx-proxy.

* Mount the nginx-conf ConfigMap to the path /etc/nginx/conf.d/default.conf in the nginx-proxy container.

* Expose the nginx-proxy deployment on port 80.

* Validate the setup by accessing the service. You should observe content from the simple-webapp response.