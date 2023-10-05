### MongoDB Deployment:

Below are the YAML manifests for creating the MongoDB PV, PVC, and Deployment:

**Step 1: Create a MongoDB PV (Persistent Volume)**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-data-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/mongo"

```
Explanation:
1. This YAML creates a Persistent Volume named mongodb-data-pv with a capacity of 10Gi.
2. It's configured for ReadWriteOnce access mode, meaning it can be mounted by a single node for both read and write operations.
3. he storage is allocated on the host at the path /mnt/mongo.

**Step 2: Create a MongoDB PVC (Persistent Volume Claim)**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10G
```

Explanation:
1. This YAML creates a Persistent Volume Claim named mongodb-data-pvc.
2. It requests storage of 10Gi with ReadWriteOnce access mode, matching the PV created earlier.

**Step 3: Create a MongoDB Deployment**

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: root
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: KodeKloudCKA
        name: mongodb
        image: mongo:4.2
        ports:
        - containerPort: 27017
        volumeMounts:
        - mountPath: /data/db
          name: data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: mongodb-data-pvc
      nodeName: worker-01 # Ensure that the MongoDB pod is deployed on a single node. (Last question)
```

Explanation:

1. This YAML creates a Deployment named mongodb-deployment with one replica.
2. It specifies that the pod should have a label app: mongodb.
3. The pod template contains a container running the mongo:4.2 image.
4. Environment variables MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD are set for MongoDB initialization.
5. The PVC created earlier (mongodb-data-pvc) is mounted at /data/db to persist data.
6. The nodeName attribute ensures that the MongoDB pod is deployed on a specific node (worker-01 in this example).

**Verification and Considerations:**

To verify that data remains intact after pod restarts, follow these steps:

1. Insert test data into the database.
2. Restart the pod.
3. Confirm that the data inserted earlier still 5. exists after the pod restarts.

Regarding the risk of data loss on pod restarts, in this configuration, since we are using a Persistent Volume Claim (PVC) backed by a Persistent Volume (PV), data is stored persistently. This means that even if the pod is restarted or rescheduled, the data will remain intact.

However, if the PV is not properly backed up, there could be a risk of data loss in the event of PV failure. To mitigate this, regular backups of the PV should be performed.


---

Good job on completing this challenge! We hope you found it informative and practical.