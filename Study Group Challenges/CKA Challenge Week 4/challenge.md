## Challenge 4

## Daemonset Configuration in Kubernetes

You are tasked with setting up a daemonset named nginx-reverse-proxy-daemonset as reverse proxy and other 2 pods service-red and service-blue. Then you need to config the nginx-reverse-proxy-daemonset to forward the path /service-red to the service-red and likewise for service-blue

1. Pod Name service-red:
 - Image: kodekloud/simple-webapp
 - Port: 8080
 - nv: APP_COLOR=red

2. Pod Name service-blue:
 -  Image: kodekloud/simple-webapp
 -  Port: 8080
 -   env: APP_COLOR=blue

3. Daemonset Name nginx-reverse-proxy:
-  Image: nginx
-  Port: 80
-  Nginx Configuration:
<img width="411" alt="image" src="https://github.com/kodekloudhub/Study-Group-Series/assets/124123623/26d0bb31-a548-4e25-b98e-385b38313fc1">


To accomplish this task, follow these steps ->

- Create a ConfigMap named nginx-conf with the provided Nginx configuration.
- Define a daemonset named nginx-reverse-proxy-daemonset.
- Mount the nginx-conf ConfigMap to the path /etc/nginx/conf.d/default.conf in the nginx-reverse-proxy container.
- Expose the nginx-reverse-proxy daemonset on port 80.
- Troubleshoot the Nginx configuration to meet the requirements.
- Validate the setup by accessing the service.



