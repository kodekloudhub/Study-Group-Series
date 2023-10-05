**Name**: John Kennedy

**Slack User ID**: U05D6BR916Z

**Email**: johnkdevops@gmail.com

---

## Solution

### Solution Details

# Question:  Ingress and Networking

## Task

## Deploy NGINX Ingress Controller using Helm:
1. Install Helm by running wget https://get.helm.sh/helm-v3.9.3-linux-amd64.tar.gz \ 
tar xvf helm-v3.9.3-linux-amd64.tar.gz \
sudo mv linux-amd64/helm /usr/local/bin \ 
rm helm-v3.4.1-linux-amd64.tar.gz \
rm -rf linux-amd64 
2. Verify helm is installed by running helm version. 
version.BuildInfo{Version:"v3.12.3", GitCommit:"3a31588ad33fe3b89af5a2a54ee1d25bfe6eaa5e", GitTreeState:"clean", GoVersion:"go1.20.7"}
3. Create new namespace 'nginx-ingress' by running kubectl create ns nginx-ingress
4. Add NGINX Ingress Repository by running helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && helm repo update
3. Install NGINX Ingress Controller using Helm by running helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace nginx-ingress \
  --set controller.replicaCount=2
4. Verify the Installation by running kubectl get pods -n nginx-ingress
NAME                                                     READY   STATUS    RESTARTS   AGE
nginx-ingress-ingress-nginx-controller-8dd55f5c9-7cb84   1/1     Running   0          29s
nginx-ingress-ingress-nginx-controller-8dd55f5c9-bxd4s   1/1     Running   0          29s

### 1: **Create a `wear-deployment`:**

Using imperative commands to create, deploy & verify the Wear Deployment:
1. Create deployment by running kubectl create deploy wear-deployment --image=kodekloud/ecommerce:apparels --port=8080 --replicas=2 --dry-run=client -o yaml > wear-deploy.yaml
2. Create the deployment by using kubectl apply -f wear-deploy.yaml.
3. Watch the deployment to see when it is ready by using kubectl get deploy -w
4. Inspect deployment by using kubectl describe deploy wear-deployment.
5. Expose the deployment run kubectl expose deploy wear-deployment --name=wear-service --port=8080 --dry-run=client -o yaml > wear-service.yaml
6. Create the service by using kubectl apply -f wear-service.yaml.
7. Verify the wear-deployment and wear-service were successfully created by running kubectl get deploy wear-deployment --show-labels && kubectl get svc wear-service -o wide
NAME              READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
wear-deployment   2/2     2            2           9m    app=wear-deployment
NAME           TYPE        CLUSTER-IP         PORT(S)    AGE   SELECTOR
wear-service   ClusterIP   10.98.110.222      8080/TCP   54s  app=wear-deployment

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
      dnsConfig:
        nameservers:
          - 10.96.0.10  # IP address of CoreDNS service
        searches:
          - cka.kodekloud.local   # Add your custom domain here
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
---

### 2: **Create a `video-deployment`:**

Using imperative commands to create, deploy & verify the Video Deployment:
1. Create deployment by running kubectl create deploy video-deployment --image=kodekloud/ecommerce:video --port=8080 --replicas=2 --dry-run=client -o yaml > video-deploy.yaml
2. Create the deployment by using kubectl apply -f video-deploy.yaml.
3. Watch the deployment to see when it is ready by using kubectl get deploy -w
4. Inspect deployment by using kubectl describe deploy video-deployment.
5. Expose the deployment run kubectl expose deploy video-deployment --name=video-service --port=8080 --dry-run=client -o yaml > video-service.yaml
6. Create the service by using kubectl apply -f video-service.yaml.
7. Verify the video-deployment and video-service were successfully created by running kubectl get deploy wear-deployment --show-labels && kubectl get svc wear-service -o wide
NAME               READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
video-deployment   2/2     2            2           58s   app=video-deployment
NAME            TYPE        CLUSTER-IP    PORT(S)    AGE   SELECTOR
video-service   ClusterIP   10.107.223.1  8080/TCP   30s   app=video-deployment

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
      dnsConfig:
        nameservers:
          - 10.96.0.10  # IP address of CoreDNS service
        searches:
          - cka.kodekloud.local   # Add your custom domain here
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
Address:          10.99.128.88
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  cka.kodekloud.local  
                       /wear    wear-service:8080 (10.244.0.10:8080,10.244.0.11:8080)
                       /video   video-service:8080 (10.244.0.12:8080,10.244.0.13:8080)
Annotations:           nginx.ingress.kubernetes.io/rewrite-target: /
Events:
  Type    Reason  Age               From                      Message
  ----    ------  ----              ----                      -------
  Normal  Sync    4s (x3 over 88s)  nginx-ingress-controller  Scheduled for sync

Note: Also, you could use imperative command to create the ingress resource by running kubectl create ingress ecommerce-ingress --rule="cka.kodekloud.local/wear=wear-service:8080" --rule="cka.kodekloud.local/video=video-service:8080" 

# YAML File

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
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

1. Find which node was your pods deployed on by running kubectl get po -o wide
2. Find out the IP address of the controlplane node by running kubectl get no controlplane -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP  
controlplane   Ready    control-plane   75m   v1.28.0   192.41.120.6
3. Find the NodePort IP of the nginx-ingress-controller svc by running kubectl describe svc -n nginx-ingress | grep -e NodePort
NodePort:                 http  30789/TCP
3. Add the virtual domain 'cka.kodekloud.local' to end of file local host file by running echo '192.39.189.3 cka.kodekloud.local:30789' >> /etc/hosts | cat /etc/hosts (This is verify the entry was added to host file)
127.0.0.1 localhost
...
...
...
192.39.189.3 cka.kodekloud.local:30789
4. Delete coredns pods by running kubectl delete po -n kube-system -l k8s-app | grep coredns
5. Verify the pods are back up by running kubectl get po -n kube-system -l k8s-app | grep coredns
coredns-5dd5756b68-gvpqx   1/1     Running   0          3m2s
coredns-5dd5756b68-jg49d   1/1     Running   0          3m2s

5. Verify can get to video and wear applications:
curl -I http://cka.kodekloud.local/wear
curl -I http://cka.kodekloud.local/video



