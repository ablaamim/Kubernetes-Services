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
kind: NetworkPolicy
metadata:
  name: full-isolation
  namespace: isolated-ns
spec:
  podSelector: {}  # Apply to all pods in the isolated-ns namespace
  policyTypes:
    - Ingress  # Controls incoming traffic
    - Egress   # Controls outgoing traffic
  ingress:
    - from:
        - namespaceSelector: {}  # Allow traffic only from the same namespace
  egress:
    - to:
        - ipBlock:
            cidr: 10.43.0.0/16  # Allow access to CoreDNS for DNS resolution
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0  # Allow outbound traffic to the internet
      ports:
        - protocol: TCP
          port: 80   # Allow HTTP
        - protocol: TCP
          port: 443  # Allow HTTPS
```

### Summury :

| **Feature** | **Behavior** |
|------------|-------------|
| **Pods inside `isolated-ns` can communicate with each other?** | 🚫 **No** |
| **Pods in `isolated-ns` can access the internet?** | ✅ **Yes** (Only HTTP/HTTPS) |
| **Pods in `isolated-ns` can resolve domain names?** | ✅ **Yes** (Allows CoreDNS) |
| **Pods in `isolated-ns` can be pinged from other namespaces?** | 🚫 **No** (Blocked by `namespaceSelector: {}`) |
