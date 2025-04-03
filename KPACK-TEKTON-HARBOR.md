## Use a Service Account with the Correct Permissions

> Create a Service Account in the namespace where your PipelineRun is running (e.g., default):

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-kpack-sa
  namespace: default
```

> Grant It Permissions to manage kpack Image resources. For example, create a ClusterRole that allows full access to kpack images:


```yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kpack-image-access
rules:
- apiGroups: ["kpack.io"]
  resources: ["images"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

> Bind the Role to your service account:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tekton-kpack-sa-binding
subjects:
- kind: ServiceAccount
  name: tekton-kpack-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: kpack-image-access
  apiGroup: rbac.authorization.k8s.io
```

## Pipeline :

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: nodejs-kpack-pipeline
spec:
  workspaces:
    - name: shared-data  # For git clone
    - name: docker-creds # For Harbor credentials

  params:
    - name: git-url
      type: string
      default: "https://github.com/heroku/node-js-sample.git"
    - name: image-tag
      type: string
      default: "10.50.29.196:30022/myproject/nodejs-app:latest"

  tasks:
    # Task 1: Clone Git repo
    - name: fetch-source
      taskRef:
        name: git-clone
      workspaces:
        - name: output
          workspace: shared-data
      params:
        - name: url
          value: "$(params.git-url)"
        - name: revision
          value: "master"

    # Task 2: Build with kpack
    - name: build-with-kpack
      runAfter: ["fetch-source"]
      workspaces:
        - name: docker-creds
          workspace: docker-creds
      params:
        - name: image-tag
          value: "$(params.image-tag)"
      taskSpec:
        steps:
          - name: trigger-build
            image: bitnami/kubectl
            script: |
              #!/bin/sh
              # Create kpack Image resource
              kubectl apply -f - <<EOF
              apiVersion: kpack.io/v1alpha2
              kind: Image
              metadata:
                name: nodejs-app
                namespace: kpack
              spec:
                tag: "$(params.image-tag)"
                builder:
                  name: default-builder-docker
                  kind: ClusterBuilder
                serviceAccountName: kpack-service-account
                source:
                  git:
                    url: "$(params.git-url)"
                    revision: "main"
                build:
                  env:
                  - name: BP_NODE_VERSION
                    value: "18.x"
                  registry:
                    insecure: true
                    insecureRegistries:
                    - "10.50.29.196:30022"
              EOF
```

## PipelineRun :

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: nodejs-kpack-pipelinerun
spec:
  serviceAccountName: tekton-kpack-sa
  podTemplate:
    securityContext:
      fsGroup: 1000   # Or use 0 if needed; adjust based on your security requirements
  pipelineRef:
    name: nodejs-kpack-pipeline
  workspaces:
    - name: shared-data
      volumeClaimTemplate:
        spec:
          accessModes: [ReadWriteOnce]
          storageClassName: longhorn
          resources:
            requests:
              storage: 1Gi
    - name: docker-creds
      secret:
	secretName: harbor-credentials
  params:
    - name: git-url
      value: "https://github.com/heroku/node-js-sample.git"
    - name: image-tag
      value: "10.50.29.196:30022/myproject/nodejs-app:latest"
```