**Name**: basil

**Slack User ID**: U05QWDXNWLF

**Email**: basil-is@superhero.true

---

## Solution

### Solution Details

Given that we are working with a deployment that relies on services external to Kubernetes, we can utilize two features to meet our needs: the readiness probe and the deployment strategy.

### Code Snippet

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: next-gen-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: next-gen-app
    spec:
      containers:
        - name: nginx-container
          image: nginx
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /index.html
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```
