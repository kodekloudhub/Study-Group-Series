**Name**: Houssein Hmila

**Slack User ID**: U054SML63K5

**Email**: housseinhmila@gmail.com   

---

## Solution

### Solution Details

#### 1. Create Deployments:

We create the wear-deployment and video-deployment using the provided specifications.
```
kubectl create deploy wear-deployment --image=kodekloud/ecommerce:apparels --port=8080 
kubectl create deploy video-deployment --image=kodekloud/ecommerce:video --port=8080
```

#### 2. Expose Deployments:

Expose both deployments to create services that can be accessed within the cluster.
```
kubectl expose deploy wear-deployment --name=wear-service --port=8080
kubectl expose deploy video-deployment --name=video-service --port=8080
```
#### 3. Configure Ingress:
We need to ensure that the DNS resolution for cka.kodekloud.local points to the IP address of your Kubernetes cluster's Ingress Controller. This can be done by adding an entry in your local DNS resolver or configuring a DNS server to resolve this domain to the cluster's IP.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx   # Specify Your Ingress class name
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
```
Now we can access our services via ``` curl cka.kodekloud.local/wear ```  or  ```  curl cka.kodekloud.local/video ```