### MongoDB Deployment Challenge

In your development environment (1 Master and 2 Workers), you require a standalone MongoDB instance running in a Kubernetes cluster to facilitate development work while conserving resources. There's no need to ensure high availability (HA) or scalability; the primary goal is to have a development database that retains data across pod restarts. This scenario outlines how to deploy MongoDB using a PV for data storage, create a PVC to claim that PV, and set environment variables for user and password configuration. The objective is to verify that data remains intact even after pod restarts.

**Tasks:**

1. **Create a MongoDB PV (Persistent Volume):**
   - Name: mongodb-data-pv
   - Storage: 10Gi
   - Access Modes: ReadWriteOnce
   - Host Path: "/mnt/mongo"

2. **Create a MongoDB PVC (Persistent Volume Claim):**
   - Name: mongodb-data-pvc
   - Storage: 10Gi
   - Access Modes: ReadWriteOnce

3. **Create a MongoDB Deployment:**
   - Name: mongodb-deployment
   - Image: mongo:4.2
   - Container Port: 27017
   - Mount mongodb-data-pvc to path /data/db
   - Env MONGO_INITDB_ROOT_USERNAME: root
   - Env MONGO_INITDB_ROOT_PASSWORD: KodeKloudCKA

4. **Verify Data Retention Across Restarts:**
   - Insert some test data into the database
   - Restart the pod
   - Confirm that the data inserted earlier still exists after the pod restarts.
   - Is there a risk of data loss when the pod is restarted, and if so, what improvement can be taken to ensure it doesn't occur?

---

*Good luck! We look forward to reviewing your solution.*