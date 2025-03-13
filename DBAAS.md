# MySQL Operator for Kubernetes

## Introduction

The MySQL Operator for Kubernetes is an operator for managing MySQL InnoDB Cluster setups inside a Kubernetes Cluster. 
It manages the full lifecycle with set up and maintenance that includes automating upgrades and backup.

MySQL Operator for Kubernetes is brought to you by the MySQL team at Oracle.

MySQL InnoDB Cluster is a high-availability, fault-tolerant MySQL deployment; the MySQL Operator handles Kubernetes-based orchestration, automating tasks like provisioning, scaling, upgrades, and failover. Together, they simplify running a production-ready, highly available MySQL database in Kubernetes.

So, with InnoDB Cluster + MySQL Operator, you “secure” (or maximize) Consistency and Partition tolerance—the “CP” corner of the CAP triangle.

### Consistency :

InnoDB Cluster uses a consensus protocol (similar to Paxos/Raft). Writes must be acknowledged by a majority of replicas, which ensures that all “up” nodes remain consistent with one another.

### Partition Tolerance :

If a network partition occurs, nodes in the “minority” partition lose write-privileges to prevent a split-brain scenario. The “majority” partition can continue accepting writes. This keeps data consistent across whichever nodes remain active.

## MySQL Operator for Kubernetes Installation on rancher K8S :

### Using Helm

Install the Helm repository:

```sh
$> helm repo add mysql-operator https://mysql.github.io/mysql-operator/
$> helm repo update
```

Then deploy the operator using the just added repository:

```sh
$> helm install mysql-operator mysql-operator/mysql-operator --namespace mysql-operator --create-namespace
```

## MySQL InnoDB Cluster Installation

### Using kubectl

For creating a MySQL InnoDB Cluster, first create a secret with credentials for a MySQL root user used to 
perform administrative tasks in the cluster. For example:

```sh
$> kubectl create secret generic mysqldbaas-secret \
        --from-literal=rootUser=root \
        --from-literal=rootHost=% \
        --from-literal=rootPassword="Mypassword"
```

Define your MySQL InnoDB Cluster, which references the secret. For example:

```yaml
apiVersion: mysql.oracle.com/v2

# ------------------------------------------------------------------------------
# 'kind' specifies which type of Kubernetes object we're creating. Here, it's 
# an 'InnoDBCluster' — a custom resource managed by the MySQL Operator. 
# ------------------------------------------------------------------------------
kind: InnoDBCluster

# ------------------------------------------------------------------------------
# 'metadata' provides identifying information for this resource, such as its name 
# and the namespace in which it lives.
# ------------------------------------------------------------------------------
metadata:
  # The human-readable name for this InnoDBCluster resource.
  name: mycluster

  # The namespace in which this InnoDBCluster will be created.
  namespace: mysql-operator

# ------------------------------------------------------------------------------
# 'spec' contains the configuration details for the InnoDBCluster. 
# ------------------------------------------------------------------------------
spec:

  # The name of the Kubernetes Secret that holds MySQL credentials or
  # other sensitive information needed for the cluster.
  secretName: mysqldbaas-secret

  # Whether to generate and use self-signed certificates for TLS encryption 
  # of MySQL connections. Useful if you don't provide a custom certificate.
  tlsUseSelfSigned: true

  # The number of MySQL instance replicas (Nodes) in the cluster.
  # Typically, 3 ensures quorum-based high availability.
  instances: 3

  # The version of MySQL to deploy in this cluster. Must be supported 
  # by the MySQL Operator's CRD.
  version: "8.0.34"

  # ----------------------------------------------------------------------------
  # 'datadirVolumeClaimTemplate' describes how persistent storage is requested
  # for MySQL data. This is used to dynamically create PersistentVolumeClaims
  # for each replica.
  # ----------------------------------------------------------------------------
  datadirVolumeClaimTemplate:

    # The name of the StorageClass that will provision the PersistentVolumes.
    storageClassName: longhorn

    # 'accessModes' defines the ways the volume can be mounted by pods.
    # Here we list one mode by default, with additional options explained below.
    accessModes:
      - ReadWriteOnce
      # ReadWriteOnce (RWO): The volume can be mounted as read-write by a single node.
      #
      # Additional Access Modes (not enabled by default in this template):
      #
      # - ReadOnlyMany (ROX): The volume can be mounted read-only by many nodes.
      #   Useful when you have multiple pods that need read access to the same data.
      #
      # - ReadWriteMany (RWX): The volume can be mounted as read-write by many nodes
      #   simultaneously. This is beneficial for workloads that require shared storage
      #   for both reading and writing across multiple pods.
      #
      # Note: The availability of these modes depends on the underlying storage provider.

    # The resource requirements for each PersistentVolumeClaim,
    # specifically how much storage is needed.
    resources:
      requests:
        # The minimum amount of disk space required for the MySQL data directory.
        storage: 4Gi
    instances: 1
```

Assuming it's saved as `mycluster.yaml`, deploy it:

```sh
$> kubectl apply -f mycluster.yaml
```

This sample creates an InnoDB Cluster with three MySQL Server instances and one MySQL Router instance. 
The process can be observed using:

```sh
$> kubectl get innodbcluster --watch

NAME          STATUS    ONLINE   INSTANCES   ROUTERS   AGE
mycluster     PENDING   0        3           1         2m6s
...
mycluster     ONLINE    3        3           1         10s
```

### Using Helm

Create MySQL InnoDB Cluster installations using defaults or with customization. 
Here's an example using all defaults for a cluster named `mycluster`:

```sh
$> helm install mycluster mysql-operator/mysql-innodbcluster
```

Or customize, this example sets options from the command line:

```sh
$> helm install mycluster mysql-operator/mysql-innodbcluster \
        --namespace mynamespace \
        --create-namespace \
        --set credentials.root.user='root' \
        --set credentials.root.password='supersecret' \
        --set credentials.root.host='%' \
        --set serverInstances=3 \
        --set routerInstances=1
```
## Execute a pod :

```bash
kubectl exec -it mycluster-0 -n dbaas-test -c mysql -- mysql -uroot -p
```