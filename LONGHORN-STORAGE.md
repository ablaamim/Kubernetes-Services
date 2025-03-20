### 📌 Structured Plan for Dynamic Pod Deployment with Optional Longhorn Storage :

🎯 Objective

* Deploy pods dynamically in Kubernetes using YAMLs.
* Optionally attach a Persistent Volume (PV) using Longhorn when required.
* Ensure that if storage is not needed, the pod runs without a volume.
* Ensure that if storage is needed, a Persistent Volume Claim (PVC) is created and mounted.

#### 🔹 Workflow :

1️⃣ Receive Deployment Request

A request is sent to deploy a pod.
The request includes or excludes a "use_storage" flag.
2️⃣ Check if Storage is Required

If no storage is required, deploy the pod without a volume.
If storage is required, proceed to the next step.
3️⃣ Create a New PVC (If Storage is Required)

A new PVC is created for the pod.
The PVC uses Longhorn storage with ReadWriteOnce (RWO) mode.
Each pod gets a unique PVC, meaning pods cannot share storage (for now).

4️⃣ Deploy the Pod

If no storage is required, deploy the basic pod YAML.
If storage is required, deploy the pod with the newly created PVC mounted.

### Manifest :

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-pvc
  namespace: my-namespace
spec:
  accessModes:
    - ReadWriteOnce  # Only one pod can mount this volume
  resources:
    requests:
      storage: 5Gi  # Request 5GB of storage
  storageClassName: longhorn  # Ensure this matches your Longhorn StorageClass
---
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  namespace: my-namespace
spec:
  containers:
    - name: my-container
      image: debian
      command: ["/bin/sh", "-c", "sleep infinity"]
      volumeMounts:
        - mountPath: "/data"
          name: my-app-volume
  volumes:
    - name: my-app-volume
      persistentVolumeClaim:
        claimName: my-app-pvc

```

### Postgresql with PVC :

```
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: postsql
  namespace: dbaas-test
spec:
  instances: 1
  storage:
    size: 2Gi
    storageClass: longhorn

```