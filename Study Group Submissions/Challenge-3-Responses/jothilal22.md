**Name**: Jothilal Sottallu

**Email**: srjothilal2001@gmail.com

---

### SOLUTION

Imagine you have a website, and you want it to be super reliable and handle lots of visitors smoothly. To achieve this, you set up a special system using Kubernetes. In this system, you have two main parts.

* First, there's the "simple-webapp," which is like the heart of your website. It knows how to show your web pages and listens for requests on the internet at port 8080.


* Think of 'nginx-proxy' as a traffic director for your website, just like a friendly hotel receptionist. It stands at the main entrance (port 80) and guides visitors to the 'simple-webapp' (port 8080), ensuring they find what they're looking for on your website smoothly.

First we will create a configmap 

```
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

* Command to apply the ConfigMap

```kubectl apply -f nginx-config.yaml```



```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy-deployment
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
            - name: nginx-conf
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: default.conf
      volumes:
        - name: nginx-conf
          configMap:
            name: nginx-conf
```
* Command to Apply the Deployment 


```kubectl apply -f nginx-proxy-deployment.yaml```

Expose the nginx-proxy deployment on port 80.

```apiVersion: v1
kind: Service
metadata:
  name: nginx-proxy-service
spec:
  selector:
    app: nginx-proxy
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort # You can choose the appropriate service type for your environment
```

* Command to Apply the Service Yaml

```kubectl apply -f nginx-proxy-service.yaml```