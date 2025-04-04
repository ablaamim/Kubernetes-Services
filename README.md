## Simlab Documentation :beginner:

<p align="center">
<img src="https://www.exekutive.biz/rekrute/file/entrepriseLogoInfo/recruiter_id/323411" width="200">
</p>

### Abstract:

This project delivers an integrated cluster management system for the College of Computing, enhancing user engagement and experience in the SIMLAB cluster. Built on Kubernetes, it enables seamless deployment of distributed systems, machine learning, big data, and batch workloads. The system embraces Infrastructure as Code (IaC) to automate infrastructure provisioning and meet diverse user needs.  

---

#### Hardware SPECS:

```

          +--------------+
          | Master Node  |
          +--------------+
                |
  ----------------------------------------------------
  |         |         |         |         |         |
+---------+ +---------+ +---------+ +---------+ +---------+ +-------------------------+
|Worker 1 | |Worker 2 | |Worker 3 | |Worker 4 | |Worker 5 | |Worker 6 (GPU: RTX A2000)|
+---------+ +---------+ +---------+ +---------+ +---------+ +-------------------------+

```

---

#### Details:

| Node       | RAM Size | CPU Count | GPU VRAM          |
|------------|----------|-----------|-------------------|
| **Master** | 32 GB    | 20        | -                 |
| Worker 1   | 16 GB    | 20        | -                 |
| Worker 2   | 8 GB     | 20        | -                 |
| Worker 3   | 16 GB    | 12        | -                 |
| Worker 4   | 24 GB    | 12        | -                 |
| Worker 5   | 8 GB     | 12        | -                 |
| Worker 6   | 32 GB    | 8         | RTX A2000 (12 GB) |
| **TOTAL**  | **122 GB** | **104**   | **12 GB**          |


---

#### Services in our cloud:

| Feature                                | Description                                                                                                           |
|----------------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| Prometheus/Grafana                     | A monitoring solution that collects, stores, and visualizes metrics from your infrastructure and applications.      |
| DBAAS: mysql inaudb / cnpostgres         | Managed database services providing MySQL and PostgreSQL deployments for reliable application data storage.           |
| Longhorn/CEPH                          | Storage platforms that offer persistent volumes, object storage, and filesystem support for Kubernetes workloads.     |
| Harbor Private Registry                | A secure container registry for storing, managing, and scanning container images with enhanced access controls.        |
| Nvidia GPU Operator / GPU Time Slicing | Tools to manage and share GPU resources, enabling efficient use of workstation GPUs via time slicing techniques.       |
| Neuvector                              | A comprehensive container security platform providing runtime protection, vulnerability scanning, and threat detection. |
| Calico/Flannel                         | Kubernetes networking solutions that facilitate pod-to-pod communication and enforce network policies across clusters. |
| Kpack                                  | An automated image builder that leverages Cloud Native Buildpacks to transform source code into container images.      |
| Tekton/Kaniko -  FLUX/ARGOCD           | CI/CD tools that build container images directly from Dockerfiles, integrating smoothly into Kubernetes pipelines.      |


### Technical Requirements:

The system must fulfill the following requirements:

#### 1.1 User Access Management:

Managing access for multiple users across various departments is crucial to ensure security and efficiency.  

#### 1.2 Computing Resource Allocation:

As the SIMLAB cluster includes CPU and GPU nodes, the system must dynamically allocate resources based on task requirements without user intervention. Testing will be conducted in a controlled environment before deployment.  

#### 1.3 User-Friendly Experience:

The system will support users with varying levels of expertise:  

- **Expert Users**: Direct CLI interaction for job creation.  
- **Intermediate Users**: Web interface with pre-configured environments.  
- **Beginner Users**: Assisted workflow for running code and datasets effortlessly.  

#### 1.4 Automation & Process Optimization:

To bridge knowledge gaps, the system integrates automated workflows via IaC, simplifying deployment and execution while improving user experience.  


## Implementation Plan for Secure Cloud Management:  

### High-Level Overview:  

This section outlines the key steps to build a secure and efficient cloud management platform with **Keycloak** for authentication and **Kubernetes** for orchestration. The focus is on **security, access control, and seamless integration** with OpenStack.

</p>
<p align="center">
<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/1*E20q5eAN6RzOXEpwHtSxXw.png" width="500">
</p>

> Kubernetes authentication keycloak oidc oauth2


---

### 1. Secure User Authentication & Authorization with Keycloak:  

- **Authentication**: Implement **OpenID Connect (OIDC)** with **Keycloak** to enforce **secure user authentication**. Keycloak acts as the **identity provider (IdP)**, centralizing user access control.  
- **Authorization**: Define **role-based access control (RBAC)** policies to restrict access based on user roles (e.g., admin, developer, viewer).  
- **Integration with Kubernetes**: Use **Keycloak tokens** for authentication within **Kubernetes clusters**, ensuring only authorized users can interact with cluster resources.  

---

### 2. Web UI Framework:

- Build a **ReactJS**-based web interface for interacting with OpenStack resources.  
- Ensure **secure session management** and **token-based authentication** using **OIDC**.  

---

### 3. Integration with OpenStack API  

- Utilize OpenStack's **RESTful APIs** to manage cloud resources (compute, networking, storage).  
- Implement **secure API communication** using **JWT tokens** and **OAuth2.0** to authenticate API requests.  

---

### 4. Secure Storage Management  

- Ensure **resource provisioning and lifecycle management** is **isolated** per user or project.  
- **Encrypt sensitive state files** and **store them securely** to prevent unauthorized access.  
- Implement **role-based storage access** for data isolation.  

---

### 5. Concurrency & State Management  

- Implement **fine-grained locking** to handle concurrent access to resources.  
- Maintain **separate, securely stored state files** per user to prevent conflicts and unauthorized modifications.  

---

### 6. Asynchronous Operations  

- Offload **resource-intensive tasks** to background jobs.  
- Use **message queues** and **event-driven workflows** to prevent UI blocking.  
- Provide **real-time progress updates** via WebSockets.  

---

### 7. Logging, Monitoring & Security Auditing  

- Implement **detailed logging** to track user actions and API interactions.  
- Use **Keycloak audit logs** for **security monitoring**.  
- Enable **Kubernetes RBAC auditing** to detect unauthorized API requests.  
- Integrate **SIEM (Security Information & Event Management)** tools for real-time threat detection.  

---

### 8. Testing & User Training  

- Conduct **penetration testing** to identify security vulnerabilities.  
- Implement **role-based testing** to ensure **secure multi-user access**.  
- Provide **user training & documentation** on best security practices.  

---

By integrating **Keycloak for authentication**, **RBAC for authorization**, and **secure API & storage management**, this solution ensures a **robust, secure, and scalable cloud management system**.


## Kubernetes Microservices Architecture

</p>
<p align="center">
<img src="https://www.opensourceforu.com/wp-content/uploads/2018/04/Figure-2-Kubernetes-architecture.jpg" width="500">
</p>

### Introduction
Kubernetes is a container orchestration platform that enables the efficient deployment, scaling, and management of containerized applications. It is widely used to implement microservices architecture, where applications are decomposed into smaller, independent services that communicate over the network.

### Key Components
1. **Kubernetes Master**: Acts as the brain of the cluster, managing scheduling, state, and orchestration of the workloads.
   - API Server: Provides a RESTful interface for interacting with Kubernetes.
   - Controller Manager: Ensures the cluster maintains its desired state.
   - Scheduler: Assigns workloads to worker nodes based on resource availability.
   - etcd: Stores configuration data and cluster state.

2. **Worker Nodes**: The physical or virtual machines that run containerized applications.
   - Kubelet: Manages containers on the node.
   - Kube Proxy: Manages networking rules and load balancing.
   - Container Runtime: Runs the container workloads (e.g., Docker, containerd).
   - Pods: The smallest deployable units, containing one or more containers.

3. **Registry**: A storage system for container images, such as Docker Hub or a private Harbor registry.

4. **Networking**: Provides communication between services via Service Meshes (e.g., Istio), CNI plugins (e.g., Calico, Flannel), and Ingress controllers.

5. **Storage**: Persistent volumes and Container Storage Interface (CSI) solutions to maintain stateful workloads.

### Microservices Workflow in Kubernetes
1. Developers push containerized microservices to the **Registry**.
2. Kubernetes **Master Node** schedules these microservices onto **Worker Nodes**.
3. Each **Worker Node** runs Pods containing microservices, exposing APIs for inter-service communication.
4. Kubernetes networking (CNI, Service Mesh) manages service discovery, routing, and load balancing.
5. Monitoring and logging tools track service performance, scaling resources when needed.

### Benefits of Kubernetes for Microservices
- **Scalability**: Auto-scaling allows services to handle varying loads efficiently.
- **Fault Tolerance**: Self-healing mechanisms restart failed containers.
- **Declarative Configuration**: Infrastructure as Code (IaC) ensures repeatability and automation.
- **Efficient Resource Management**: Schedulers optimize workload distribution.
- **Security & RBAC**: Role-Based Access Control (RBAC) and service mesh policies secure communications.

### Conclusion
Kubernetes provides a robust environment for running microservices, enabling organizations to build highly scalable and resilient applications. By leveraging Kubernetes features such as service discovery, automated scaling, and self-healing, businesses can efficiently manage complex distributed systems.

