## RESSOURCE GROUP IN RKE2 :

```
Geographical & Logical Hierarchy in Rancher Kubernetes
───────────────────────────────────────────────────────────

Physical Infrastructure (Geographical)
    │
    ├── Data Center 1 (Region/Location)
    │      ├── Kubernetes Cluster-A
    │      │      ├── Node-01 (Worker node)
    │      │      └── Node-02 (Worker node)
    │      │
    │      └── Kubernetes Cluster-B
    │             ├── Node-03 (GPU Node)
    │             └── Node-04
    │
    └── Data Center 2 (Another Region/Location)
           └── Kubernetes Cluster-C
                  ├── Node-05
                  └── Node-06
                      └── ... (additional nodes)

Kubernetes Cluster (Managed by Rancher)
    │
    ├── Project: Team-X (Logical grouping of namespaces, quotas, RBAC)
    │      ├── Namespace: Frontend
    │      │      ├── Workloads (Deployments, Pods)
    │      │      ├── Services, Ingress
    │      │      └── PersistentVolumeClaims (PVCs)
    │      │
    │      └── Namespace: Backend
    │             ├── Workloads (Deployments, Pods)
    │             └── Additional Resources (Services, Secrets, PVCs)
    │
    └── Project: Team-Y (Target specific nodes via nodeSelectors)
           ├── Namespace: GPU-Compute
           │       └── Deployment with NodeSelector → Node-03 (GPU node)
           │
           └── Namespace: Monitoring
                   └── Deployment with NodeSelector → Node-04

Access Control (RBAC - Permissions)
    │
    ├── Cluster-level RBAC
    │      └── Cluster Admin (manage all Projects/Namespaces)
    │
    ├── Project-level RBAC
    │      ├── Project Owner (manage all namespaces in the Project)
    │      └── Project Member (specific permissions across Project namespaces)
    │
    └── Namespace-level RBAC
           ├── Namespace Admin (full permission within namespace)
           ├── Namespace Developer (create, modify, delete workloads)
           └── Namespace Viewer (read-only access)


```

Explanation of Hierarchy Levels:

Physical Infrastructure (Geographical):
* Represents physical location and hardware.

Kubernetes Clusters:
* Kubernetes control-plane and nodes grouped physically or logically by location or purpose.

Nodes:
* Physical or virtual machines where pods actually run. Nodes are labeled to control workload scheduling.

Projects (Rancher-specific):
* Logical grouping for organizing namespaces and assigning permissions, policies, and resource quotas.

Namespaces:
* Kubernetes-level logical grouping to isolate resources.

Resources (Deployments, Pods, Services):
* Actual applications and workloads running in the namespace, controlled via node selectors or tolerations.

RBAC:
* Permissions from cluster-wide to namespace-specific for precise control and security.