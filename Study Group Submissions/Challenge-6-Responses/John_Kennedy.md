**Name**: John Kennedy

**Slack User ID**: U05D6BR916Z

**Email**: johnkdevops@gmail.com

---

## Solution

### Solution Details

# Question:  Ingress and Networking

## Task

## Deploy NGINX Ingress Controller using Helm:
1. Install Helm by running wget https://get.helm.sh/helm-v3.10.3-linux-amd64.tar.gz && \
tar -zxf helm-v3.10.3-linux-amd64.tar.gz && \
sudo mv linux-amd64/helm /usr/local/bin 
2. Cleanup installation files by running rm helm-v3.10.3-linux-amd64.tar.gz && \
rm -rf linux-amd64 
2. Verify helm is installed by running helm version. 
version.BuildInfo{Version:"v3.10.3", GitCommit:"3a31588ad33fe3b89af5a2a54ee1d25bfe6eaa5e", GitTreeState:"clean", GoVersion:"go1.18.9"}
3. Create new namespace 'nginx-ingress' by running kubectl create ns ingress-nginx
4. Add NGINX Ingress Repository by running helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && helm repo update
4. Install NGINX Ingress Controller using Helm by running helm install ingress-nginx \
    --set controller.service.type=NodePort \
    --set controller.service.nodePorts.http=30080 \
    --set controller.service.nodePorts.https=30443 \
    --repo https://kubernetes.github.io/ingress-nginx \
    ingress-nginx
5. Verify the Installation by running kubectl get deploy
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           2m12s

### 1: **Create a `wear-deployment`:**

Using imperative commands to create, deploy & verify the Wear Deployment:
1. Create deployment by running kubectl create deploy wear-deployment --image=kodekloud/ecommerce:apparels --port 8080 --dry-run=client -o yaml > wear-deploy.yaml
2. Create the deployment by using kubectl apply -f wear-deploy.yaml.
3. Watch the deployment to see when it is ready by using kubectl get deploy -w
4. Inspect deployment by using kubectl describe deploy wear-deployment.
5. Expose the deployment run kubectl expose deploy wear-deployment --name=wear-service --type=ClusterIP --port=8080 --target-port=8080 --dry-run=client -o yaml > wear-service.yaml
6. Create the service by using kubectl apply -f wear-service.yaml.
7. Verify the wear-deployment and wear-service were successfully created by running kubectl get deploy wear-deployment --show-labels && kubectl get svc wear-service -o wide && kubectl get ep | grep -e wear
NAME              READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
wear-deployment   1/1     1            1           9m    app=wear-deployment
NAME           TYPE        CLUSTER-IP         PORT(S)    AGE   SELECTOR
wear-service   ClusterIP   10.103.41.162      8080/TCP   54s  app=wear-deployment
NAME                                 ENDPOINTS                      AGE
ingress-nginx-controller             10.244.0.5:443,10.244.0.5:80   9m1s
ingress-nginx-controller-admission   10.244.0.5:8443                9m1s
wear-service                         10.244.0.7:8080                6s

# YAML File
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wear-deployment
  name: wear-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wear-deployment
  template:
    metadata:
      labels:
        app: wear-deployment
    spec:
      containers:
      - name: wear-container
        image: kodekloud/ecommerce:apparels
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: wear-deployment
  name: wear-service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: wear-deployment
  type: ClusterIP
---

### 2: **Create a `video-deployment`:**

Using imperative commands to create, deploy & verify the Video Deployment:
1. Create deployment by running kubectl create deploy video-deployment --image=kodekloud/ecommerce:video --port=8080 --dry-run=client -o yaml > video-deploy.yaml
2. Create the deployment by using kubectl apply -f video-deploy.yaml.
3. Watch the deployment to see when it is ready by using kubectl get deploy -w
4. Inspect deployment by using kubectl describe deploy video-deployment.
5. Expose the deployment run kubectl expose deploy video-deployment --name=video-service --type=ClusterIP --port=8080 --target-port=8080 --dry-run=client -o yaml > video-service.yaml
6. Create the service by using kubectl apply -f video-service.yaml.
7. Verify the video-deployment and video-service were successfully created by running kubectl get deploy wear-deployment --show-labels && kubectl get svc wear-service -o wide
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
video-deployment   1/1     1            1           58s   app=video-deployment
NAME                      TYPE        CLUSTER-IP    PORT(S)          SELECTOR    AGE   
ingress-nginx-controller  NodePort    10.96.176.190 80:30080/TCP,443:30443/TCP   19m
video-service             ClusterIP   10.96.4.39    8080/TCP         app=video-deployment
NAME                                 ENDPOINTS                      AGE
ingress-nginx-controller             10.244.0.5:443,10.244.0.5:80   18m
ingress-nginx-controller-admission   10.244.0.5:8443                18m
video-service                        10.244.0.8:8080                11s

# YAML File

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: video-deployment
  labels:
    app: video-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: video-deployment
  template:
    metadata:
      labels:
        app: video-deployment
    spec:
      containers:
      - name: video-container
        image: kodekloud/ecommerce:apparels
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: video-deployment
  name: video-service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: video-deployment
  type: ClusterIP
---

### 3: **Create an Ingress:**

1. Go to https://kubernetes.io/docs/home/ and search for Ingress.
2. Search for hostname and copy service-networking-ingress-wildcard-host-yaml file 
4. Create a blank yaml file for the ingress resource by running vi ecommerce-ingress.yaml, :set paste, and, paste yaml the content.
5. Make the changes to the yaml file and save it by using :wq!
6. Create the ingress resource with dry run by running kubectl apply -f ecommerce-ingress.yaml --dry-run=client -o yaml to verify yaml file is correct
7. Remove dry run option, apply, and verify that ingress resource was created successfully by running the following imperative commands:
kubectl describe ing ecommerce-ingress
Name:             ecommerce-ingress
Labels:           <none>
Namespace:        default
Address:          
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  cka.kodekloud.local  
                       /wear    wear-service:8080 (10.244.0.7:8080)
                       /video   video-service:8080 (10.244.0.8:8080)
Annotations:           nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  Sync    9s    nginx-ingress-controller  Scheduled for sync

Note: Also, you could use imperative command to create the ingress resource by running kubectl create ingress ecommerce-ingress --dry-run=client -o yaml --annotation nginx.ingress.kubernetes.io/rewrite-target=/ --rule cka.kodekloud.local/wear*=wear-service:8080 --rule cka.kodekloud.local/video*=video-service:8080 > ingress.yaml 

# YAML File

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx                 # Add
  rules:
  - host: cka.kodekloud.local
    http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port:
              number: 8080
      - path: /video
        pathType: Prefix
        backend:
          service:
            name: video-service
            port:
              number: 8080
---

4. **Point virtual domain `cka.kodekloud.local` to your kubernetes cluster and test it.**

1. Find out the IP address of the controlplane node by running kubectl get no controlplane -o wide
NAME           STATUS   ROLES           AGE     VERSION   INTERNAL-IP  
controlplane   Ready    control-plane   4h55m   v1.28.0   192.26.65.8

2. Find out the ingress NodePort number by running kubectl get svc | grep -e NodePort
NodePort: http  30080/TCP
3. Add the virtual domain 'cka.kodekloud.local' to end of file local host file by running echo '192.26.65.8 cka.kodekloud.local' >> /etc/hosts | cat /etc/hosts (This is verify the entry was added to host file)
127.0.0.1 localhost
...
...
...
192.26.65.8 cka.kodekloud.local

4. Verify can get to wear and video applications:
Wear Application:

curl -I cka.kodekloud.local:30080/wear 
HTTP/1.1 200 OK
Date: Sat, 07 Oct 2023 23:07:14 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 296
Connection: keep-alive

curl -H "Host: cka.kodekloud.local" 192.26.65.8:30080/wear
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #2980b9;">

<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">
    <img src="https://res.cloudinary.com/cloudusthad/image/upload/v1547052428/apparels.jpg">

</div>

</body>

Video Application:

curl -I cka.kodekloud.local:30080/video
HTTP/1.1 200 OK
Date: Sat, 07 Oct 2023 23:09:09 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 293
Connection: keep-alive

curl -H "Host: cka.kodekloud.local" 192.26.65.8:30080/video
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #30336b;">

<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">
    <img src="https://res.cloudinary.com/cloudusthad/image/upload/v1547052431/video.jpg">

</div>

</body>

Tests all passed!!


