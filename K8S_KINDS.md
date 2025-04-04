## Kubernetes Resource Kinds (Summary Table)

| Kind               | Purpose / Functionality                                             | Example Usage                                   |
|--------------------|---------------------------------------------------------------------|-------------------------------------------------|
| **Pod**            | Smallest deployable unit, runs one or more containers               | Deploying a single instance of an application   |
| **Deployment**     | Manages multiple replicas of pods; supports rolling updates         | Scaling web applications, updating apps smoothly|
| **Service**        | Provides network access to pods (load balancing, service discovery) | Exposing applications internally or externally  |
| **Ingress**        | Manages external HTTP(S) traffic routing to services                | Routing external web traffic to multiple apps   |
| **ConfigMap**      | Stores non-sensitive configuration data                             | Injecting app configs, feature flags            |
| **Secret**         | Securely stores sensitive data (passwords, keys, tokens)            | Database credentials, API keys                  |
| **PersistentVolume** (PV) | Storage resource provisioned by an admin; persists data beyond pod lifecycle | Long-term storage for databases, apps           |
| **PersistentVolumeClaim** (PVC) | Requests storage (PV) for pods or deployments               | Pods claiming persistent storage (databases)    |
| **Namespace**      | Logical isolation & grouping of resources                           | Separate dev/test/prod environments             |
| **ServiceAccount** | Authenticates pods accessing the Kubernetes API                     | Allowing pods to interact securely with the API |
| **StatefulSet**    | Manages stateful apps with stable identities & persistent storage   | Databases (MySQL, Cassandra), message queues    |
| **DaemonSet**      | Runs one instance of a pod on every node                            | Node-level monitoring, log collectors           |
| **Job**            | Runs short-lived, one-off tasks                                     | Database migrations, batch processing tasks     |
| **CronJob**        | Runs scheduled jobs repeatedly (cron schedule)                      | Scheduled backups, nightly data processing      |