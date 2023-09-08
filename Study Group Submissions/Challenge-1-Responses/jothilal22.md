**Name**: Jothilal Sottallu

**Email**: srjothilal2001@gmail.com

---

### SOLUTION

#### SETUP

Minikube is a tool that lets you run a single-node Kubernetes cluster locally on your computer.It's designed for development, testing, and learning purposes, allowing you to create, manage, and deploy Kubernetes applications without the need for a full-scale production cluster. 

**INSTALL A HYPERVISOR** ~ DOCKER DESKTOP [With Kubernetes Enabled]

**INSTALL A MINIKUBE** ~ You can download and install Minikube from the official GitHub repository or by using a package manager like Homebrew (on macOS)

**START MINIKUBE** ~ Open a Terminal and run the command:
**minikube start**

**WAIT FOR SETUP** ~ Minikube will download the necessary ISO image, create a virtual machine, and configure the Kubernetes cluster. This might take a few minutes depending on your internet speed.

**Stop Minikube** ~ When you're done using the Minikube cluster, you can stop it with:
**minikube stop**

#### SOLUTION DETAILS

Task: Created deployment configurations for two applications using the nginx:latest image.

Deployment for Production:
* Created the deployment configuration for the Production application in the non-production environment.
* Applied the deployment using the command: kubectl apply -f production-app.yaml

Pod Status Check:
* Checked the status of pods using kubectl get pods.
* Observed that the development application pod was in a "Pending" state, indicating it hadn't started properly.

**ERROR**
```
->   Warning  FailedScheduling  31s   default-scheduler  0/1 nodes are available: 1 node(s) didn't match Pod's node affinity/selector. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling..
```

Node Assignment:
* Examined the details of the pending pod and noted that it hadn't been assigned to a node yet.
* Updated the Minikube node using label node, which labeled the node as suitable for prod.

```
kubectl lable node minikube environment=production
 node/minikube labeled
```

Pod Status Update:
1. After updating the node label, checked pod status again using kubectl get pods.

2. Found that the development application pod was now running successfully, indicating that it was scheduled on a node

Log Analysis:
* Inspected the logs of the running pod to gain insight into its startup process.
```
EXAMPLE: One of the pod name 
kubectl logs -f production-app-c58d79c56-2g577 

```
* Observed that the nginx server was starting up, and worker processes were being initialized.

## CODE SNIPPET

PRODUCTION DEPLOYMENT

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
spec:
  selector:
    matchLabels:
      app: prod-app
  replicas: 3
  template:
    metadata:
      labels:
        app: prod-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - production
      containers:
        - image: nginx:latest
          name: nginx

```
POD LOGS 

```
└─(20:57:07 on main ✭)──> kbl -f production-app-c58d79c56-2g577
+kubectl logs
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/08/30 15:26:43 [notice] 1#1: using the "epoll" event method
2023/08/30 15:26:43 [notice] 1#1: nginx/1.25.2
2023/08/30 15:26:43 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2023/08/30 15:26:43 [notice] 1#1: OS: Linux 5.15.49-linuxkit-pr
2023/08/30 15:26:43 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/08/30 15:26:43 [notice] 1#1: start worker processes
2023/08/30 15:26:43 [notice] 1#1: start worker process 29
2023/08/30 15:26:43 [notice] 1#1: start worker process 30
2023/08/30 15:26:43 [notice] 1#1: start worker process 31
2023/08/30 15:26:43 [notice] 1#1: start worker process 32
2023/08/30 15:26:43 [notice] 1#1: start worker process 33

```



## CHALLENGE 2
#### SOLUTION DETAILS

Task 2: Propose a solution to ensure high availability of the service during and after deployments. Consider the following requirements:

Minimize the occurrence of 503 - Service Unavailable errors during deployments.
Ensure that the service is only considered ready when all its required dependencies are also ready

Liveness Probe : The liveness probe is responsible for determining whether a container is alive or dead. If the liveness probe fails (returns an error), Kubernetes will restart the container.
During deployment, liveness probes quickly detect and replace containers with issues.
* It can indirectly help prevent 503 errors by ensuring that containers that are unresponsive or in a failed state are restarted promptly. If a container becomes unresponsive, it's replaced, reducing the chances of a long-lasting 503 error.

Readiness Probe: The readiness probe is responsible for determining whether a container is ready to receive incoming network traffic. If the readiness probe fails (returns an error), the container is not included in the pool of endpoints for a Service.
* Readiness probes are crucial during deployment to confirm that a container is ready before it gets traffic. They prevent routing requests to containers that are still initializing or not fully functional.
* It prevent 503 errors during deployment by blocking traffic to unready containers. If a container is still initializing, it won't receive requests until it's ready, preventing 503 errors from unprepared containers.

## CODE SNIPPET

```
Production Deployment

apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
spec:
  selector:
    matchLabels:
      app: prod-app
  replicas: 3
  template:
    metadata:
      labels:
        app: prod-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - production
      containers:
        - image: nginx:latest
          name: nginx
          ports:
            - containerPort: 80  

          # Readiness probe
          readinessProbe:
            failureThreshold: 3
          httpGet:
            path: /actuator/health/readiness
            port: 80
            scheme: HTTP
          periodSeconds: 20
          successThreshold: 1
          timeoutSeconds: 60
          initialDelaySeconds: 60        

          # Liveness probe
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness 
              port: 80
            initialDelaySeconds: 60  
            periodSeconds: 10   
            timeoutSeconds: 60
            successThreshold: 1
            failureThreshold: 3    

```
