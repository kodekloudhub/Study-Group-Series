**Name**: Adam Jurkiewicz

**Slack User ID**: U05PNQU3CUA

**Email**: a.jurkiewicz5 (at) gmail.com

---

## Solution

### Solution Details

First, create an `nginx.conf` file and paste the provided nginx configuration. Next, the configuration can be applied imperatively:

```bash
$ kubectl create cm nginx-conf --from-file default.conf
```

Secondly, create a Deployment manifest using `kubectl create deployment` client dry-run. Container names will be customized after generating the manifest.

```bash
$ kubectl create deploy nginx-proxy-deployment --image kodekloud/simple-webapp --image nginx -o yaml --dry-run=client > nginx-proxy-deployment.yml
```

The Deployment should look as follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-proxy-deployment
  name: nginx-proxy-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-proxy-deployment
  template:
    metadata:
      labels:
        app: nginx-proxy-deployment
    spec:
      containers:
      - image: kodekloud/simple-webapp
        name: simple-webapp
        ports:
          - containerPort: 8080
      - image: nginx
        name: nginx-proxy
        ports:
          - containerPort: 80
        volumeMounts:
        - name: nginx-conf-volume
          mountPath: /etc/nginx/conf.d
      volumes:
        - name: nginx-conf-volume
          configMap:
            name: nginx-conf

```

Apply the Deployment manifest file using `kubectl apply`:

```bash
$ kubectl apply -f nginx-proxy-deployment.yml
```

The Deployment can then be exposed with `kubectl expose`:

```bash
$ kubectl expose deploy nginx-proxy-deployment --port 80
```

### Testing

Tests can be made within a `curlimages/curl` Pod:

```
$ kubectl run -it curlpod --image curlimages/curl -- curl -v nginx-proxy-deployment
```
