**Name**: Jothilal Sottallu

**Email**: srjothilal2001@gmail.com


### SOLUTION


### TASK: 

1. Create a MongoDB PV (Persistent Volume):
Name: mongodb-data-pv
Storage: 10Gi
Access Modes: ReadWriteOnce
Host Path: "/mnt/mongo"

   * Persistent Volume 
       * Think of a Persistent Volume as a reserved space on a Persistent Disk. It's like reserving a section of that virtual hard drive for a specific purpose. This way, you can use it later to store your files or data.

##### PV YAML
```
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
    path: /mnt/mongo
```
Apply this configuration to Kubernetes cluster using Kubectl command

```kubectl apply -f pv.yaml```


2. Create a MongoDB PVC (Persistent Volume Claim):
Name: mongodb-data-pvc
Storage: 10Gi
Access Modes: ReadWriteOnce
Create a MongoDB Deployment:


    * Persistent Volume Claim
        * A Persistent Volume Claim is like saying, "Hey, I need some space on that virtual hard drive." It's a request you make when you want to use some storage for your application. When you make this request, the system finds a suitable place on a Persistent Disk and sets it aside for you to use.

##### PVC YAML

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Apply this configuration to Kubernetes cluster using Kubectl command

```kubectl apply -f pvc.yaml```
 
3. Name: mongodb-deployment
Image: mongo:4.2
Container Port: 27017
Mount mongodb-data-pvc to path /data/db
Env MONGO_INITDB_ROOT_USERNAME: root
Env MONGO_INITDB_ROOT_PASSWORD: KodeKloudCKA

#### Deployment YAML

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
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
        - name: mongodb
          image: mongo:4.2
          ports:
            - containerPort: 27017
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              value: root
            - name: MONGO_INITDB_ROOT_PASSWORD
              value: KodeKloudCKA
          volumeMounts:
            - name: mongo-data
              mountPath: /data/db
      volumes:
        - name: mongo-data
          persistentVolumeClaim:
            claimName: mongodb-data-pvc

```
Apply this configuration to Kubernetes cluster using Kubectl command

```kubectl apply -f deployment.yaml```


4. Verify Data Retention Across Restarts:
Insert some test data into the database
Restart the pod
Confirm that the data inserted earlier still exists after the pod restarts.
Is there a risk of data loss when the pod is restarted, and if so, what improvement can be taken to ensure it doesn't occur?

* I inserted the data to check when we restarting the pods do we have save data or not 
* kubectl delete pod deployment-x8tsd

When a pod restarts in Kubernetes, the data within the attached Persistent Volume (PV) should be retained, assuming the PV is properly configured and the data is persisted within 

### RISK OF DATA LOSS 
1. Ensure that data is always persisted within the PVC or PV. MongoDB data should be stored in a path that is mounted from the PV, as configured in your MongoDB container.
2. Use appropriate storage classes and reclaim policies to retain PVs when pods are deleted. For example, use the "Retain" policy to prevent automatic PV deletion.