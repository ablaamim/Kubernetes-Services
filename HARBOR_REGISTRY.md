## HARBOR PRIVATE REGISTRY:

Harbor is an open source trusted cloud native registry project that stores, signs, and scans content. Harbor extends the open source Docker Distribution by adding the functionalities usually required by users such as security, identity and management. Having a registry closer to the build and run environment can improve the image transfer efficiency. Harbor supports replication of images between registries, and also offers advanced security features such as user management, access control and activity auditing.

Harbor is hosted by the Cloud Native Computing Foundation (CNCF). If you are an organization that wants to help shape the evolution of cloud native technologies, consider joining the CNCF. For details about whose involved and how Harbor plays a role, read the CNCF announcement.

<p align="center">
<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*uNg7Q4hoFBWT2YBRR9qSyw.png" width="1200">
</p>

## INSTALLATION:

### Custom Values File To Use http:


```yaml
# Harbor Configuration using NodePort Service Type
# ----------------------------------------------

# Service Exposure Configuration
# -----------------------------
expose:
  type: nodePort  # Use NodePort service type instead of Ingress for direct node access
  
  nodePort:
    name: harbor  # Name for the NodePort service
    ports:
      http:
        port: 80  # Internal container port for HTTP
        nodePort: 30022  # External NodePort for HTTP access (accessible at <NodeIP>:30022)
      https:
        port: 443  # Internal container port for HTTPS (unused when TLS disabled)
        nodePort: 30443  # Reserved NodePort for HTTPS (can be used if enabling TLS later)

  # TLS/SSL Configuration
  # ---------------------
  tls:
    enabled: false  # Disable HTTPS/TLS to use plain HTTP
    # Note: When TLS is disabled, the HTTPS NodePort won't be functional
    # The following TLS certificate settings can be uncommented if enabling TLS later:
    # certSource: secret  # Certificate source type
    # secret:
    #   secretName: harbor-tls  # Name of the Kubernetes secret containing TLS certs

# External Access Configuration
# ----------------------------
externalURL: "http://10.50.29.196:30022"  # Full external access URL using:
                                           # - HTTP protocol
                                           # - Node IP address (10.50.29.196)
                                           # - HTTP NodePort (30022)

# Persistent Storage Configuration
# -------------------------------
persistence:
  enabled: true  # Enable persistent storage
  persistentVolumeClaim:
    registry:
      storageClass: "longhorn"  # Use Longhorn storage class
      size: "50Gi"  # 50GB for container image storage
    jobservice:
      storageClass: "longhorn"
      size: "1Gi"  # 1GB for job service data
    redis:
      storageClass: "longhorn"
      size: "1Gi"  # 1GB for Redis caching
    trivy:
      storageClass: "longhorn"
      size: "5Gi"  # 5GB for vulnerability scanning (if enabled)
    database:
      storageClass: "longhorn"
      size: "5Gi"  # 5GB for PostgreSQL database

# Component Configuration
# ----------------------
notary:
  enabled: false  # Disable Notary (image signing service)

chartmuseum:
  enabled: false  # Disable ChartMuseum (Helm chart repository)

# Administrator Credentials
# ------------------------
harborAdminPassword: "StrongPasswordHere"  # Initial admin password
                                           # CHANGE THIS IN PRODUCTION!
                                           # Must meet complexity requirements

```

## Add Harbor Helm Repository:

```bash
helm repo add harbor https://helm.goharbor.io
helm repo update

```
## Create Namespace:

```bash
kubectl create namespace harbor
```

## Install Harbor:

```bash
helm install harbor harbor/harbor \
  -n harbor \
  -f harbor-nodeport-values.yaml
```

##  Verify Installation:

```bash
  # Check pods (wait for all to be 'Running')
kubectl get pods -n harbor -w

# Check services (verify NodePort assignments)
kubectl get svc -n harbor

# Check persistent volume claims
kubectl get pvc -n harbor
```

## INTEGRATION WITH RANCHER WITHOUT TLS:

### How This Configuration Avoids TLS:

* Protocol Specification:

The http:// prefix in the endpoint URL explicitly tells containerd to use unencrypted HTTP

This overrides any default HTTPS behavior

* Missing TLS Components:

No certificate authority (ca_file) is specified

No TLS certificates are configured in Harbor's Helm values (tls.enabled: false)

* NodePort Service:

Harbor is configured to expose port 30022 as a NodePort for HTTP traffic

No HTTPS port (443) is actively used in this setup

* Verification Disabled:

While insecure_skip_verify: true is present, it has no effect with HTTP

### CHANGES DONE IN RANCHER:

```yaml
# /etc/rancher/rke2/registries.yaml - Harbor Registry Configuration
# -----------------------------------------------------------------

# (not active)
# mirrors:
#   "10.50.29.196:30022":
#     endpoint:
#       - "http://10.50.29.196:30022"
#
# configs:
#   "10.50.29.196:30022":
#     tls:
#       ca_file: "/etc/rancher/rke2/certs/harbor-ca.crt"  # Would be needed for HTTPS
#       insecure_skip_verify: true  # Would skip cert validation if using self-signed certs



# Active Configuration (HTTP)
# ---------------------------
mirrors:
  "10.50.29.196:30022":
    endpoint:
      - "http://10.50.29.196:30022"  # Explicit HTTP protocol forces non-TLS communication

configs:
  "10.50.29.196:30022":
    auth:
      username: admin
      password: StrongPasswordHere  # Basic auth over HTTP (not encrypted)
    tls:
      insecure_skip_verify: true  # Irrelevant for HTTP but would bypass validation if using HTTPS
```

### RESTARTING THE RANCHER SERVER IS MANDATORY TO APPLY CHANGES:

```
systemctl restart rke2-server
```

### Secret creation : 

kubectl delete secret docker-credentials --ignore-not-found

# Create new secret with proper formatting

```yaml
kubectl create secret generic docker-credentials \
  --from-file=config.json=<(echo '{
    "auths": {
      "http://10.50.29.196:30022": {
        "auth": "'$(echo -n "admin:StrongPasswordHere" | base64)'",
        "email": "abdessamad.laamimi@um6p.ma"
      }
    },
    "HttpHeaders": {
      "User-Agent": "Docker-Client/19.03.12 (linux)"
    }
  }')
```

### View in YAML Format

```bash
kubectl get secret harbor-credentials -o yaml
```

### Decode :

```bash
echo "<encoded-data>" | base64 -d
```

