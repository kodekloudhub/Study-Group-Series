***Name***: Vinoth Subbiah
***Email***: vinoji2005@hotmail.com

____________________________________________________________________________________________
***Solution :***
***Step 1: Create Pods for service-red and service-blue**

***service-red.yaml:***
```bash
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
---
apiVersion: v1
kind: Service
metadata:
  name: service-red
spec:
  selector:
    app: service-red
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

***service-blue.yaml:***
```bash 
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
---
apiVersion: v1
kind: Service
metadata:
  name: service-blue
spec:
  selector:
    app: service-blue
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
***Step 2: Create a ConfigMap for Nginx Configuration***
***Create a ConfigMap named nginx-conf with the Nginx configuration to route traffic.***
***nginx-conf.yaml:***
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  default.conf: |
    server {
        listen 80;
        location /service-red {
            proxy_pass http://service-red:80;
        }
        location /service-blue {
            proxy_pass http://service-blue:80;
        }
    }
```
***Step 3: Create the DaemonSet***
***Create the DaemonSet named nginx-reverse-proxy-daemonset using the Nginx image and mount the ConfigMap.***

***nginx-reverse-proxy-daemonset.yaml:***
```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-reverse-proxy-daemonset
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
            - name: nginx-config-volume
              mountPath: /etc/nginx/conf.d/default.conf
              subPath: default.conf
      volumes:
        - name: nginx-config-volume
          configMap:
            name: nginx-conf
```
***Step 4: Apply the Configurations***
***Now, apply the configurations using kubectl***
```bash
kubectl apply -f service-red.yaml
kubectl apply -f service-blue.yaml
kubectl apply -f nginx-conf.yaml
kubectl apply -f nginx-reverse-proxy-daemonset.yaml
```
***Step 5: Verify the Setup***
***To verify that everything is set up correctly, you can check the pods, services, and daemonset.***

```bash
kubectl get pods
kubectl get svc
kubectl get daemonset
```

***Step 6: Access the Services***
***To validate the setup, you can use port forwarding to access the Nginx reverse proxy and test the routing.***

```bash
kubectl port-forward daemonset/nginx-reverse-proxy-daemonset 8080:80
```
***Now, you can access the services using the following URLs in your browser or using curl:***

```bash 
http://localhost:8080/service-red
http://localhost:8080/service-blue
```
***These URLs should route traffic to the respective services***

