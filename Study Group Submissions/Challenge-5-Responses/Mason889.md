**Name**: Kostiantyn Zaihraiev

**Slack User ID**: U04FZK4HL0L

**Email**: kostya.zaigraev@gmail.com

---

## Solution

### Solution Details

After reading task, I've decided to create yaml templates that will be used to easy configuration and management of all resources.

I've create `persistentvolume.yaml` and `persistentvolumeclaim.yaml` files and after that applied them to the cluster:

```bash
kubectl apply -f persistentvolume.yaml
kubectl apply -f persistentvolumeclaim.yaml
```

To check that everything is ok I've used `kubectl get pv` and `kubectl get pvc` commands:

```bash
kubectl get pv

NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                      STORAGECLASS   REASON   AGE
mongodb-data-pv   10Gi       RWO            Retain           Bound    default/mongodb-data-pvc   standard                7s

kubectl get pvc

NAME               STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mongodb-data-pvc   Bound    mongodb-data-pv   10Gi       RWO            standard       2s
```

Then I've predefined `deployment.yaml` file using `kubectl` command:

```bash
kubectl create deployment mongodb-deployment --image=mongo:4.2 --dry-run=client -o yaml > deployment.yaml
```

After making some changes in the file I've applied it to the cluster:

```bash
kubectl apply -f deployment.yaml
```

(And validated that everything is ok using `kubectl get pods` command)
```bash
kubectl get pods

NAME                                  READY   STATUS    RESTARTS   AGE
mongodb-deployment-5fbc48f59d-nng6w   1/1     Running   0          3m12s
```

To check that everything is working correctly I've connected to the pod and inserted some data:

```bash
kubectl exec -it mongodb-deployment-5fbc48f59d-nng6w -- mongo -u root -p KodeKloudCKA
---

> db.createCollection("test")
{ "ok" : 1 }
> db.test.insert({"name": "test"})
WriteResult({ "nInserted" : 1 })
> exit
bye
```

Then I've restarted the pod and checked that data is still there:

```bash
kubectl delete pod mongodb-deployment-5fbc48f59d-nng6w

kubectl get pods

NAME                                  READY   STATUS    RESTARTS   AGE
mongodb-deployment-5fbc48f59d-z4cmv   1/1     Running   0          11s
```

And checked that data is still there:

```bash
kubectl exec -it mongodb-deployment-5fbc48f59d-z4cmv -- mongo -u root -p KodeKloudCKA
---

> db.test.find()
{ "_id" : ObjectId("6515c16f91cea01bdf14c2e5"), "name" : "test" }
> exit
bye
```

And as you can see data is still there, so persistent volume claim works correctly.

### Code Snippet

#### persistentvolume.yaml
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-data-pv
spec:
  capacity:
    storage: 10Gi
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/mongo
```

#### persistentvolumeclaim.yaml
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-data-pvc
spec:
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
```


#### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
spec:
  selector:
    matchLabels:
      app: mongodb-deployment
  template:
    metadata:
      labels:
        app: mongodb-deployment
    spec:
      containers:
      - name: mongodb-container
        image: mongo:4.2
        resources: {}
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: root
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: KodeKloudCKA
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
      volumes:
      - name: mongodb-data
        persistentVolumeClaim:
          claimName: mongodb-data-pvc
```
