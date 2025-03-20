##  Network Isolation :

How Rancher Handles Network Policies Internally
Rancher does not enforce NetworkPolicies directly but relies on the CNI (Container Network Interface) used in the cluster to enforce them.
 
the default CNI is Canal (Calico + Flannel), which means:

Flannel handles basic networking (pod-to-pod connectivity).
Calico handles NetworkPolicy enforcement.

### 📌 Keys :

* Rancher manages NetworkPolicies through Kubernetes APIs.
* Enforcement is done by Calico in RKE2.
* Rancher UI provides an interface to manage NetworkPolicies, but it does not apply them directly.

### 📌  How NetworkPolicies Work in Rancher (RKE2)

1️⃣ Network Traffic Flow
When a NetworkPolicy is created in Rancher (or kubectl), it follows this workflow:

You create a NetworkPolicy (kubectl apply -f policy.yaml).
Rancher stores it in the Kubernetes API (networking.k8s.io/v1).
Calico reads the policy and applies rules using iptables/BPF.
Flannel ensures pod connectivity, but only for allowed traffic.
Traffic is filtered at the node level, enforcing policy rules.

2️⃣  Policy Enforcement: Calico’s Role
Calico applies iptables rules per node to allow/deny traffic based on NetworkPolicies.
Each pod gets an IP address (Flannel) but is subject to Calico's policies.
Calico’s Felix agent syncs policies from Kubernetes and updates networking rules.

📌 How to Modify Rancher’s Network Policy Behavior

RKE2 uses Calico’s Global Network Policies, which can override standard Kubernetes policies. Example:

```
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: deny-all  # Name of the policy

spec:
  selector: all()  # Apply this policy to all pods in the cluster

  types:
    - Ingress  # Block all incoming traffic
    - Egress   # Block all outgoing traffic

  ingress: []  # No allowed inbound traffic (full isolation)
  egress: []   # No allowed outbound traffic (full isolation)
```

### 📌 Custom policy :

This Kubernetes NetworkPolicy is designed to isolate all pods in the namespace (isolated-ns) from each other while still allowing outbound internet access.

```
apiVersion: networking.k8s.io/v1
# Specifies the Kubernetes API version used for the NetworkPolicy resource. 
# Here, it uses the stable "networking.k8s.io/v1" API.

kind: NetworkPolicy
# Declares that this resource is a NetworkPolicy, which is used to control network access.

metadata:
  name: full-isolation
  # The name "full-isolation" is an identifier for this policy within the namespace.
  namespace: isolated-ns
  # The policy is applied to the "isolated-ns" namespace. Only pods in this namespace are affected.

spec:
  podSelector: {}
  # An empty podSelector means that this policy applies to all pods in the "isolated-ns" namespace.
  # If you wanted to target a specific set of pods, you'd provide label selectors here.

  policyTypes:
    - Ingress
    - Egress
  # "policyTypes" defines the directions of traffic the policy applies to.
  # In this case, it controls both incoming (Ingress) and outgoing (Egress) traffic.

  ingress: []
  # An empty ingress rule list means no incoming traffic is allowed to the pods.
  # This effectively isolates the pods from any external (or even intra-namespace) connections.
  
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
  # The egress section specifies what outbound connections are permitted.
  # "ipBlock" with "cidr: 0.0.0.0/0" means that all IP addresses are allowed as destinations.
  # Because no specific ports are defined, all protocols and ports are permitted for outbound traffic.

```

### Summury :

| **Feature** | **Behavior** |
|------------|-------------|
| **Pods inside `isolated-ns` can communicate with each other?** | 🚫 **No** |
| **Pods in `isolated-ns` can access the internet?** | ✅ **Yes** (Only HTTP/HTTPS) |
| **Pods in `isolated-ns` can resolve domain names?** | ✅ **Yes** (Allows CoreDNS) |
| **Pods in `isolated-ns` can be pinged from other namespaces?** | 🚫 **No** (Blocked by `namespaceSelector: {}`) |

### Test :

```
  GNU nano 5.6.1                                                                                    debian-pods.yaml                                                                                               
apiVersion: v1
kind: Pod
metadata:
  name: debian-pod-1
  namespace: isolated-ns
  labels:
    app: debian1
spec:
  containers:
    - name: debian
      image: debian
      command: ["/bin/sh", "-c", "sleep infinity"]

---
apiVersion: v1
kind: Pod
metadata:
  name: debian-pod-2
  namespace: isolated-ns
  labels:
    app: debian2
spec:
  containers:
    - name: debian
      image: debian
      command: ["/bin/sh", "-c", "sleep infinity"]
```

### 

```
$> python3 -m http.server 8080  # for instance
$> curl http://pod-ip:8080
```
