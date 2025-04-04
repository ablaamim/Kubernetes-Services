## Tekton Overview

> Tekton is an open-source, cloud-native framework for creating Continuous Integration/Continuous Delivery (CI/CD) pipelines in Kubernetes.
> It enables teams to define reusable pipeline components declaratively using Kubernetes-native resources, such as Custom Resource Definitions (CRDs).


<p align="center">
<img src="https://baptistout.net/kub-native-ci-cd/cncf-bingo.webp1" width="200">
</p>


---

## Example Scenarios for Tekton:

* Building Docker container images from source code and pushing them to private registries.

* Running unit and integration tests inside Kubernetes pods.

* Deploying applications using Helm, Kustomize, or kubectl after successful builds.

* Automating machine-learning model deployments into production.

---

✅ Complete Workflow Explained :

Pipeline Triggered:

Clones the repo from GitHub into a volume (shared-data).

Build and Push:

Kaniko builds the Docker image (simple-python-app:latest).

Uses credentials from the provided Docker Secret to authenticate with the private registry.

Pushes the final built image to registry at 10.50.29.196:30022.

-> Result:

A Docker image is built from the specified git repository and pushed automatically, all within Kubernetes, fully automated and Kubernetes-native.



## Kaniko yaml :

> Kaniko is an open-source tool for building and pushing container images from within a Kubernetes cluster, without needing Docker installed. Tekton integrates Kaniko as a Task to build container images in pipelines.

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

---

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

---

## Pipelinerun

```yaml
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