## Sharing A Nvidia GPU Between Pods In Kubernetes :

[DOC](https://dev.to/thenjdevopsguy/sharing-a-nvidia-gpu-between-pods-in-kubernetes-4hp9)

## GPU Slicing 💡


When you hear “slicing”, it’s a method of taking one GPU and allowing it to be used across more than one Pod.

There’s also a method called MPS, but it seems like slicing is used the most right now.

The first thing you’ll do is set up a Config Map for the slicing.

Once thing to point out is the replica count. Notice how the replica count currently says 4? That means four (4) Pods can use the GPU. If you bumped it up to ten, that means ten (10) Pods could share the GPU. This of course depends on the type of GPU and if the resources are available like any other piece of hardware.

## configmap :

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