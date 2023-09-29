**Name**: Houssein Hmila

**Slack User ID**: U054SML63K5

**Email**: housseinhmila@gmail.com   

---

## Solution

### Solution Details
To achieve the task of deploying MongoDB in a Kubernetes cluster with persistent storage and verifying data retention across pod restarts, we can follow these steps:

#### 1. Create a MongoDB PV (Persistent Volume):
```vim pv.yaml```
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-data-pv
spec:
  hostPath:
    path: /mnt/mongo
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  ```
  #### 2. Create a MongoDB PVC (Persistent Volume Claim):

```vim pvc.yaml```
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi
  storageClassName: manual
  ```
 after apply this two YAML files, you should see your pv and pvc in bound status!.

#### 3. Create a MongoDB Deployment:
 ```
 k create deployment mongodb-deployment --image=mongo:4.2 --port=27017 --dry-run=client -o yaml > mongo.yaml 
 ```

after add the env variables and volumeMount our file should be like this:

```yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: mongodb-deployment
  name: mongodb-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb-deployment
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: mongodb-deployment
    spec:
      containers:
      - image: mongo:4.2
        name: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: "root"
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: "KodeKloudCKA"
        volumeMounts:
        - mountPath: "/data/db"
          name: mypd
      volumes:
      - name: mypd
        persistentVolumeClaim:
          claimName: mongodb-data-pvc
```
Now access your pod using:
```
kubectl exec -ti <new_pod_name> -- sh
```
then connect to our mongo database:
```
mongo --username root --password KodeKloudCKA
use mydatabase
```
finally insert your data:
```
db.mycollection.insertOne({
    name: "Houssein Hmila",
    linkedin: "https://www.linkedin.com/in/houssein-hmila/",
    email: "housseinhmila@gmail.com"
})
```
now we delete the pod (it will be another pod deployed)
-> we can access the new pod and reconnect to our mongo database and then check the data inserted before using this command:
```
db.mycollection.find()
```
you will get something like this: 

{ "_id" : ObjectId("651464aebd39fc98c777a3dd"), "name" : "Houssein Hmila", "linkedin" : "https://www.linkedin.com/in/houssein-hmila/", "email" : "housseinhmila@gmail.com" }

##### so now we are sure that our data persist. 

 #### To improve data resilience, We can use cloud-based solutions like Amazon EBS (Elastic Block Store) or equivalent offerings from other cloud providers.