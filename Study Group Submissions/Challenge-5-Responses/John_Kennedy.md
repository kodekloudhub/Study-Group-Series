**Name**: John Kennedy

**Slack User ID**: U05D6BR916Z

**Email**: johnkdevops@gmail.com

---

## Solution

### Solution Details

# Question:  MongoDB Deployment Challenge

## Task

### 1: **Create a MongoDB PV (Persistent Volume):**

1. Go to https://kubernetes.io/docs/home/ and search for PersistentVolumes.
2. Choose Configure a Pod to Use a PersistentVolume for Storage.
3. Search for PersistentVolume and copy pods/storage/pv-volume.yaml file.
4. Create a blank yaml file for the persistent volume by running vi mongodb.pv.yaml, :set paste, and, paste yaml the content.
5. Make the changes to the yaml file and save it by using :wq!
6. Create the persistent volume with dry run by running kubectl apply -f mongodb.pv.yaml --dry-run=client -o yaml to verify yaml file is correct
7. Remove dry run option, apply, and verify that persistent volume was created successfully by running the following imperative commands:
kubectl get pv mongodb-data-pv -o wide
kubectl describe pv mongodb-data-pv 

# YAML File

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

### 2: **Create a MongoDB PVC (Persistent Volume Claim):**

1. Go to https://kubernetes.io/docs/home/ and search for PersistentVolumes.
2. Choose Configure a Pod to Use a PersistentVolume for Storage.
3. Search for PersistentVolumeClaim and copy pods/storage/pv-claim.yaml file.
4. Create a blank yaml file for the persistent volume by running vi mongodb.pvc.yaml, :set paste, and, paste yaml the content.
5. Make the changes to the yaml file and save it by using :wq!
6. Create the persistent volume with dry run by running kubectl apply -f mongodb.pvc.yaml --dry-run=client -o yaml to verify yaml file is correct
7. Remove dry run option, apply, and verify that persistent volume was created successfully by running the following imperative commands:
kubectl get pv mongodb-data-pvc -o wide
kubectl describe pvc mongodb-data-pvc 

# YAML File

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

## Verify that PeristentVolumeClaim is bound to PersistentVolume
NAME                  CAPACITY ACCESS MODES RECLAIM POLICY STATUS                            
pv***/mongodb-data-pv 10Gi     RWO          Retain         Bound           
                          
NAME                                     STATUS   VOLUME            CAPACITY   ACCESS MODES   
persistentvolumeclaim/mongodb-data-pvc   Bound    mongodb-data-pv   10Gi       RWO

### 3: **Create a MongoDB Deployment:**

1. Go to https://kubernetes.io/docs/home/ and search for PersistentVolumes.
2. Choose Configure a Pod to Use a PersistentVolume for Storage.
3. Search for Create a Pod and keep the kubernetes doc for reference.

 Using imperative commands to create, deploy & verify the MongoDB Deployment:
1. Create deployment by running kubectl create deploy mongodb-deployment --image=mongo:4.2 --port= 27017 --replicas=2 --dry-run=client -o yaml > mongodb-deploy.yaml
2. Open the .yaml file by runing vi mongodb-deploy.yaml. add persistentvolume claim,volume,and environment variables.
3. Search for Enviromental and keep kubernetes doc for reference.
4. Add both environmental variables to mongodb container.
1. Add volumeMounts section name: mongodb-pvc-volume with mountPath: "/data/db".
2. Add volumes section with name of volume: name: mongodb-pvc-volume with persistentVolumeClaim:mongodb-data-pvc.
3. Create the deployment by using kubectl apply -f mongodb-deploy.yaml.
4. Watch the deployment to see when it is ready by using k get deploy -w
5. Inspect deployment by using k describe deploy nginx-proxy-deployment.

# YAML File

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: mongodb-deployment
  name: mongodb-deployment
spec:
  replicas: 1 # You can adjust the number of replicas as needed
  selector:
    matchLabels:
      app: mongodb-deployment
  template:
    metadata:
      labels:
        app: mongodb-deployment
    spec:
      volumes:                                           
        - name: mongodb-pvc-volume                       # Add
          persistentVolumeClaim:                         # Add
            claimName: mongodb-data-pvc                  # Add
      containers:
        - name: mongodb                                  # Change from mongo
          image: mongo:4.2
          env:                                           # Add
          - name: MONGO_INITDB_ROOT_USERNAME             # Add
            value: "root"                                # Add
          - name: MONGO_INITDB_ROOT_PASSWORD             # Add
            value: "KodeKloudCKA"                        # Add
          ports:
            - containerPort: 27017
          volumeMounts:                                  # Add
            - mountPath: "/data/db"                      # Add
              name: mongodb-pvc-volume                   # Add
      
### Expose the mongodb deployment on ClusterIP. 

1. Expose the deployment with dry run by using kubectl expose deploy mongodb-deployment --name=mongodb-service --port=27017 --target-port=27017 --dry-run=client -o yaml > mongodb-svc.yaml.
2. Create ClusterIP service for the deployment using kubectl apply -f mongodb-svc.yaml.
3. I would verify the svc was created for the deployment by using kubectl get svc mongodb-service -o wide.

## YAML File

apiVersion: v1
kind: Service
metadata:
  labels: 
    app: mongodb-deployment
  name: mongodb-service
spec:
  ports:
  - port: 27017
    protocol: TCP
    targetPort: 27017
  selector:
    app: mongodb-deployment
  

### 4: **Verify Data Retention Across Restarts:**

1. Connect to running pod by running kubectl exec -it mongodb-deployment-84dcb59d94-crmrd -- bash
2. Connect to mongodb by running mongo -u root and added password: KodeKloudCKA
3. To see a list of database run show dbs
4. Create or Use a database run use data
5. Show current database run db
6. Create a persons collection run db.createCollection("persons")
7. Insert data into persons collection run:
db.persons.insert({"name": "KodeKloudDevOps1", "experience": "Beginner",})
8. Verify data was inserted run db.persons.find()
{ "_id" : ObjectId("651492156f6daed11b03dfd6"), "name" : "KodeKloudDevOps1", "experience" : "Beginner" }
9. Exit out of mongodb and po by running Ctrl +C and exit.
8. Delete po by running kubectl delete po mongodb-deployment-84dcb59d94-crmrd.
9. Watch for po to coming up by running kubectl get po -w.
10. Once the pod is back up, connect to it by running kubectl exec -it mongodb-deployment-84dcb59d94-crmrd -- mongo -u root -p KodeKloudCKA
11. Select data run use data.
12. Select collection run use persons
13. Verify row of data you inserted is still available run db.persons.find()
{ "_id" : ObjectId("651492156f6daed11b03dfd6"), "name" : "KodeKloudDevOps1", "experience" : "Beginner" }

## Is there a risk of data loss when the pod is restarted, and if so, what improvement can be taken to ensure it doesn't occur?

There could be if the node is not available.
Here are a couple of improvements could be taken:
1. Increase the replicas of your deployment and use NodeAffinity to make sure the pods are placed on the right size node if your demands spikes.
2. Create kubectl batch job to backup the mongodb database regularly in case of emergencies or maintenance. 



