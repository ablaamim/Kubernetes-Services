## CREATE SECRET :

```bash
kubectl create secret docker-registry kpack-secret-registry   --docker-server=10.50.29.196:30022   --docker-username=admin   --docker-password=StrongPasswordHere   --docker-email=abdessamad.laamimi@um6p.ma
```

## SERVICE ACCOUNT :

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: harbor-service-account
  namespace: default
secrets:
  - name: kpack-secret-registry

```

## IMAGE BUILD : 

```yaml
apiVersion: kpack.io/v1alpha2
kind: Image
metadata:
  name: nodejs-app-image
  namespace: default
spec:
  tag: 10.50.29.196:30022/library/my-app:latest
  serviceAccountName: harbor-service-account
  source:
    git:
      url: "https://github.com/heroku/node-js-getting-started.git"
      revision: main
  builder:
    kind: ClusterBuilder
    name: default-builder-docker
```