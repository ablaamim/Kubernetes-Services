### What is Rook?

<p align="center">
<img src="https://ksingh7.medium.com/rook-ceph-deployment-on-openshift-4-2b34dfb6a442g" width="800">
</p>


Rook is an open source cloud-native storage orchestrator for Kubernetes, providing the platform, framework, and support for Ceph storage to natively integrate with Kubernetes.

Ceph is a distributed storage system that provides file, block and object storage and is deployed in large scale production clusters.

Rook automates deployment and management of Ceph to provide self-managing, self-scaling, and self-healing storage services. The Rook operator does this by building on Kubernetes resources to deploy, configure, provision, scale, upgrade, and monitor Ceph.

The status of the Ceph storage provider is Stable. Features and improvements will be planned for many future versions. Upgrades between versions are provided to ensure backward compatibility between releases.

Rook is hosted by the Cloud Native Computing Foundation (CNCF) as a graduated level project. If you are a company that wants to help shape the evolution of technologies that are container-packaged, dynamically-scheduled and microservices-oriented, consider joining the CNCF. For details about who's involved and how Rook plays a role, read the CNCF announcement.

### How to custom install :

```bash
git clone --single-branch --branch v1.10.13 https://github.com/rook/rook.git
cd rook/deploy/examples
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
```

### Custom cluster yaml :

```yaml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    image: quay.io/ceph/ceph:v19.2.1
  dataDirHostPath: /var/lib/rook
  mon:
    count: 1
    allowMultiplePerNode: true
  dashboard:
    enabled: true
  placement:
    all:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
              - gpu-worker
  storage:
    useAllNodes: false
    nodes:
    - name: gpu-worker
      devices:
      - name: "sda"

```

## BUCKET PROVISIONING :

```yaml
# my-bucket-obc.yaml
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: my-bucket-1
  namespace: haitham-bensaghir-namespace
spec:
  generateBucketName: hamza-bucket
  storageClassName: rook-ceph-bucket
```

```bash
kubectl get secret my-bucket-1 -n hamza-lachkar-namespace -o yaml
kubectl get configmap my-bucket-1 -n hamza-lachkar-namespace -o yaml
```

✅ Test Rook-Ceph Buckets with AWS CLI

## check version :

```bash
aws --version
```

🔑 To retrieve your S3 access and secret keys :

```bash
ACCESS_KEY=$(kubectl -n rook-ceph get secret rook-ceph-object-user-my-store-my-user -o jsonpath="{.data.AccessKey}" | base64 --decode)
SECRET_KEY=$(kubectl -n rook-ceph get secret rook-ceph-object-user-my-store-my-user -o jsonpath="{.data.SecretKey}" | base64 --decode)

echo "Access Key: $ACCESS_KEY"
echo "Secret Key: $SECRET_KEY"
```

✅ Next: Test with AWS CLI
Now you can use those keys with:

```bash
aws configure --profile rook-ceph
```

And then:

```bash
aws --endpoint-url http://10.50.29.196:30500 --profile rook-ceph s3 ls
```

🪣 Create Buckets & Upload Files (via AWS CLI)

✅ Create a bucket

```bash
aws --endpoint-url http://10.50.29.196:30500 --profile rook-ceph s3 mb s3://mybucket
```

✅ Upload a file

```bash
echo "Hello Rook!" > hello.txt
aws --endpoint-url http://10.50.29.196:30500 --profile rook-ceph s3 cp hello.txt s3://mybucket/
```

✅ List files

```bash
aws --endpoint-url http://<node-ip>:30500 --profile rook-ceph s3 ls s3://mybucket/
```
