**Name**: John Kennedy

**Slack User ID**: U05D6BR916Z

**Email**: johnkdevops@gmail.com

---

## Solution

### Solution Details

# Question: Deployment Configuration in Kubernetes

## Task

### Create a ConfigMap named nginx-conf with the provided Nginx configuration.

1. I used the kubernetes docs to get a sample configmap YAML file
2. I create a yaml file being using vi nginx.conf.yaml.
3. I made changes to name to nginx-conf, added key nginx:.
4. I added nginx configuration info to value field of YAML file.
5. I ran dry-run before creating the configMap by using k apply -f nginx-conf.yaml --dry-run=client -o yaml to verify the YAML file.
6. I created the configMap by using k apply -f nginx-conf.yaml
7. I inspected the configMap by using k describe cm nginx.conf

## YAML File

apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    server {
      listen 80;
      server_name localhost;
      location / {
          proxy_set_header   X-Forwarded-For $remote_addr;
          proxy_set_header   Host $http_host;
          proxy_pass         "http://127.0.0.1:8080";
      }
    }

### Define a deployment named nginx-proxy-deployment with two containers: simple-webapp and nginx-proxy.

1. I used imperative commands to create deployment by running k create deploy nginx-proxy-deployment --image=kodekloud/simple-webapp port=8080 --dry-run=client -o yaml > nginx-proxy.deploy.yaml
2. I opened the nginx-proxy-deploy.yaml file by using vi nginx-proxy-deployment and added the nginx-proxy container.

## YAML File

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy-deployment
spec:
  replicas: 1 # You can adjust the number of replicas as needed
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
        - name: nginx-proxy        # Add 
          image: nginx             # Add
          ports:                   # Add
            - containerPort: 80    # Add


### Mount the nginx-conf ConfigMap to the path /etc/nginx/conf.d/default.conf in the nginx-proxy container.

1. I added volumeMounts section name: nginx-conf-volume with mountPath: /etc/nginx/conf.d/default.conf subPath: nginx.conf to nginx-proxy container.
2. I added volumes section with name of volume: name: nginx-conf-volume with configMap: name: nginx-conf.
3. I create the deploy by using k apply -f nginx-proxy-deploy.yaml
4. I watch the deploy to see when it is ready by using k get deploy -w
5. I inspect deployment by using k describe deploy nginx-proxy-deployment.

## YAML File

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-proxy-deployment
spec:
  replicas: 1 # You can adjust the number of replicas as needed
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
          volumeMounts:                                  # Add
            - name: nginx-conf-volume                    # Add
              mountPath: /etc/nginx/conf.d/default.conf  # Add
              subPath: nginx.conf                        # Add
      volumes:                                           # Add
        - name: nginx-conf-volume                        # Add
          configMap:                                     # Add
            name: nginx-conf                             # Add

### Expose the nginx-proxy deployment on port 80.

1. I expose the deployment with dry run by using k expose deploy nginx-proxy-deployment --name=nginx-proxy-service --type=NodePort --port=80 --target-port=80 --dry-run=client -o yaml > nginx-proxy-svc.yaml
2. I would expose the deployment using k apply -f nginx-proxy-svc.yaml
3. I would verify the svc was created for the deployment by using k get svc nginx-proxy-service -o wide
4. I will note what is the NodePort to use: 30576

## YAML File

apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: nginx-proxy-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx-proxy
  type: NodePort

### Validate the setup by accessing the service. You should observe content from the simple-webapp response.

1. Verify I can get the the simple-webapp application I use curl -I http://localhost:30576
2. I get the 200 OK
3. Verify I can get to nginx-proxy by using curl -I http://localhost:8080.
4. I get the 200 OK
5. Everything up running and working!!




