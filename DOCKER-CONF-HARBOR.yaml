**Docker Configuration for Private Registry**

This document outlines the steps and configuration snippets you need to connect your Docker client to a private registry served over HTTP or HTTPS, including:

1. Marking a registry as **insecure** (HTTP) in the Docker daemon
2. Trusting a self-signed certificate for TLS
3. Logging in, tagging, and pushing images

---

## 1. Docker Daemon Configuration (`/etc/docker/daemon.json`)

To allow Docker to communicate over plain HTTP (NodePort registry), add the registry under `insecure-registries`.

```json
{
  "insecure-registries": [
    "10.50.29.205:30022"
  ]
}
```

- **File path:** `/etc/docker/daemon.json`
- **Action:** Create or edit this file to include your registry host:port.

### Reload and Restart Docker

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Verify Docker picked up the setting:

```bash
docker info | grep -i insecure
# Should list 10.50.29.205:30022
```

---

## 2. Trusting a Self-Signed Certificate (HTTPS)

If your registry uses a self-signed certificate, Docker can trust it via `certs.d`:

1. Create the directory for your registry’s host:port:

   ```bash
   sudo mkdir -p /etc/docker/certs.d/10.50.29.205:30022
   ```

2. Copy your CA certificate into that directory and name it `ca.crt`:

   ```bash
   sudo cp /path/to/registry.crt \
     /etc/docker/certs.d/10.50.29.205:30022/ca.crt
   ```

3. Restart Docker:

   ```bash
   sudo systemctl restart docker
   ```

After this, Docker will verify TLS when connecting to your registry.

---

## 3. Logging In, Tagging & Pushing

1. **Log in** to the registry (replace credentials as needed):

   ```bash
   docker login 10.50.29.205:30022
   # Enter username/password when prompted
   ```

2. **Tag** your local image into a Harbor project (e.g., `library`):

   ```bash
   docker tag local-image:tag 10.50.29.205:30022/library/local-image:tag
   ```

3. **Push** the image:

   ```bash
   docker push 10.50.29.205:30022/library/local-image:tag
   ```

If you have a custom project (e.g., `myproject`), replace `library` with your project name.

---

## 4. Sample Workflow

```bash
# 1. Configure Docker daemon
# (edit /etc/docker/daemon.json, then restart)

# 2. (Optional) Trust self-signed cert
sudo mkdir -p /etc/docker/certs.d/10.50.29.205:30022
sudo cp registry.crt /etc/docker/certs.d/10.50.29.205:30022/ca.crt
sudo systemctl restart docker

# 3. Log in
docker login 10.50.29.205:30022

# 4. Tag & push
docker tag debian:latest 10.50.29.205:30022/library/debian:new
docker push 10.50.29.205:30022/library/debian:new
```

---

**Notes:**

- Use `insecure-registries` for HTTP or `certs.d` for TLS. Do not mix both.
- Ensure the registry service is reachable and the Harbor project exists before pushing.
- On RKE2 nodes, you can also configure containerd’s trust as shown previously.

*End of document.*

