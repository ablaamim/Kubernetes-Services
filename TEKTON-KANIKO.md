## Verify Harbor Access Manually

```bash
# Test Harbor API access
curl -v -u admin:StrongPasswordHere http://10.50.29.196:30022/api/v2.0/projects/myproject

# Try to push a test image manually
docker login 10.50.29.196:30022 -u admin -p yourpassword
docker pull alpine
docker tag alpine 10.50.29.196:30022/myproject/alpine-test
docker push 10.50.29.196:30022/myproject/alpine-test
```

## Create a New Docker Config Secret Properly

```bash
# Delete existing secret if any
kubectl delete secret docker-credentials --ignore-not-found

# Create new secret with proper formatting
kubectl create secret generic harbor-credentials \
  --from-file=config.json=<(echo '{
    "auths": {
      "http://10.50.29.196:30022": {
        "auth": "'$(echo -n "admin:StrongPasswordHere" | base64)'",
        "email": "admin@example.com"
      }
    },
    "HttpHeaders": {
      "User-Agent": "Docker-Client/19.03.12 (linux)"
    }
  }')
```

## Kaniko yaml :

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: kaniko-insecure
spec:
  workspaces:
    - name: source
    - name: dockerconfig
  params:
    - name: IMAGE
      type: string
    - name: CONTEXT
      type: string
      default: "."
    - name: DOCKERFILE
      type: string
      default: "Dockerfile"
  steps:
    - name: build-and-push
      image: gcr.io/kaniko-project/executor:v1.9.1
      workingDir: $(workspaces.source.path)
      env:
        - name: DOCKER_CONFIG
          value: "/workspace/dockerconfig"
      command:
        - /kaniko/executor
      args:
        - --dockerfile=$(params.DOCKERFILE)
        - --context=$(params.CONTEXT)
        - --destination=$(params.IMAGE)
        - --insecure
        - --insecure-registry
        - --skip-tls-verify
      workspaces:
        - name: source
          mountPath: $(workspaces.source.path)
        - name: dockerconfig
          mountPath: /workspace/dockerconfig
```

## Pipeline :

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: clone-build-push
  namespace: default
spec:
  description: |
    This pipeline clones a git repo, builds a Docker image with Kaniko, and pushes it to a registry.
  params:
    - name: repo-url
      type: string
    - name: image-reference
      type: string
  workspaces:
    - name: shared-data
    - name: docker-credentials
  tasks:
    - name: fetch-source
      taskRef:
        name: git-clone
      workspaces:
        - name: output
          workspace: shared-data
      params:
        - name: url
          value: $(params.repo-url)
    - name: build-push
      runAfter: ["fetch-source"]
      taskRef:
        name: kaniko
      workspaces:
        - name: source
          workspace: shared-data
        - name: dockerconfig
          workspace: docker-credentials
      params:
        - name: IMAGE
          value: $(params.image-reference)
        - name: context
          value: "."
        - name: dockerfile
          value: "Dockerfile"
        - name: insecure
          value: "true"
        - name: skip-tls-verify
          value: "true"
        - name : insecure_pull
          value : "true"

```

## Pipelinerun

```
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: clone-build-push-run
spec:
  pipelineRef:
    name: clone-build-push
  podTemplate:
    securityContext:
      fsGroup: 65532
  workspaces:
    - name: shared-data
      volumeClaimTemplate:
        spec:
          accessModes: [ReadWriteOnce]
          storageClassName: longhorn
          resources:
            requests:
              storage: 1Gi
    - name: docker-credentials
      secret:
        secretName: docker-credentials
        items:
          - key: config.json
            path: config.json
  params:
    - name: repo-url
      value: "https://github.com/ablaamim/Kaniko-Test.git"
    - name: image-reference
      value: "10.50.29.196:30022/myproject/simple-python-app:latest"

```