## Sharing A Nvidia GPU Between Pods In Kubernetes :

<p align="center">
<img src="https://github.com/ablaamim/SIMLAB-IAC/blob/main/images/ts.png" width="1200">
</p>


## GPU time Slicing 💡

NVIDIA Time Slicing in Kubernetes allows multiple pods to share a single GPU by dividing the GPU's time into slices that are allocated to different pods. This is implemented through the NVIDIA Device Plugin and Kubernetes resource allocation mechanisms.

## Summury:

| Aspect                      | NVIDIA MIG (Multi-Instance GPU)                       | GPU Time-Slicing                                |
|-----------------------------|-------------------------------------------------------|-------------------------------------------------|
| **Resource Partitioning**   | Hardware-level (physical partitions)                  | Software-level (logical time-based sharing)     |
| **Isolation Level**         | Strong isolation (dedicated memory & compute slices)  | Weak isolation (shared GPU memory & compute)    |
| **Performance Stability**   | High, predictable performance (fixed resources)       | Variable; potential contention & overhead       |
| **Use-Case Suitability**    | GPU-intensive, latency-sensitive workloads            | Lightweight workloads, dev/test scenarios       |
| **Configuration Complexity**| Medium complexity (MIG profiles)                      | Higher complexity (driver & scheduler config)   |
| **Resource Efficiency**     | Good (resources partitioned, but fixed allocations)   | Very High (maximizes GPU utilization)           |
| **Flexibility**             | Lower (predefined MIG profiles, fixed slices)         | Higher (dynamic, workload-based GPU allocation) |
| **Management Overhead**     | Lower (static management once configured)             | Higher (dynamic scheduling management required) |

## How NVIDIA GPU Time-Slicing Works in Kubernetes:

GPU Time-Slicing is a software-based method allowing multiple pods to share a single GPU by scheduling short time intervals ("slices") of GPU execution to different workloads.

Instead of giving each pod exclusive GPU access, the NVIDIA GPU driver rapidly switches execution between multiple pods, effectively giving each pod the illusion of exclusive GPU access, but in reality, sharing the GPU hardware.

##  Causes of NVIDIA GPU Time-Slicing Failures:

| Scenario                          | Reason for Failure                            | Result / Symptoms                      |
|-----------------------------------|-----------------------------------------------|----------------------------------------|
| GPU-intensive workloads           | High contention for GPU resources             | Performance degradation, latency spikes|
| Real-time, latency-critical apps  | Context switching overhead                    | Increased latency, missed deadlines    |
| Memory-intensive workloads        | GPU memory exhaustion                         | OOM errors, crashes, unstable pods     |
| Misconfigured GPU scheduler       | Incorrect scheduler or driver settings        | Pods not launching, unstable GPU usage |
| Apps needing exclusive GPU access | Applications assuming dedicated GPU           | Crashes, unpredictable behavior        |
| Driver incompatibility            | Outdated/incompatible NVIDIA drivers          | GPU instability, kernel panics         |


## configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: plugin-config
  namespace: gpu-operator
data:
  time-slicing: |-
    version: v1
    flags:
      migStrategy: none
    sharing:
      timeSlicing:
        renameByDefault: false
        resources:
          - name: nvidia.com/gpu
            replicas: 4
  mps: |-
    version: v1
    flags:
      migStrategy: none
    sharing:
      mps:
        renameByDefault: false
        resources:
          - name: nvidia.com/gpu
            replicas: 4
```

Next, patch the cluster policy to have the ability to implement Nvidia’s time slicing.

```bash
cat > patch.yaml << EOF
spec:
  devicePlugin:
    config:
      name: plugin-config
      default: time-slicing
EOF

kubectl patch clusterpolicies.nvidia.com/cluster-policy --type=merge --patch-file=patch.yaml
```