# Docker & Kubernetes — Complete Technical Revision Notes

> A practical reference guide for containerization, orchestration, deployment, networking, storage, and cloud-native engineering.

---

# 1. Containerization Fundamentals

## What is containerization?

Containerization is a technology that packages an application along with its dependencies, libraries, runtime, and configuration into an isolated environment called a **container**.

It ensures that an application behaves consistently across different environments.

### Why do we need it?

Consider an application that works perfectly on a developer's Mac but fails on another developer's machine because of:

- Different Node.js or Python versions.
- Missing dependencies.
- Different operating systems.
- Configuration mismatches.
- Different installed software.

Containers help eliminate these environment-related inconsistencies.

### What is a container?

A container is an isolated running process that uses the host operating system's kernel.

It includes:
- Application code.
- Runtime.
- Libraries.
- Dependencies.
- Required configuration.

### Container vs Virtual Machine

| Virtual Machine | Container |
|---|---|
| Virtualizes hardware | Provides OS-level isolation |
| Runs a guest OS | Shares host kernel |
| Generally heavier | Generally lightweight |
| Takes more resources | Usually uses fewer resources |
| Boots a guest OS | Starts isolated processes |
| Uses a hypervisor | Uses a container runtime |

**Remember:** Containers are isolated environments, not complete virtual machines.

## Important container concepts

| Concept | Meaning |
|---|---|
| Image | Read-only template used to create containers |
| Container | Running instance of an image |
| Registry | Stores and distributes container images |
| Docker Engine | Builds and manages containers |
| Runtime | Executes containers |
| Docker Hub | Public container image registry |
| Dockerfile | Instructions for building an image |
| Docker Compose | Defines and runs multi-container applications |

---

# 2. Docker

## What is Docker?

Docker is a containerization platform that enables developers to build, package, distribute, and run applications inside containers.

It simplifies application deployment by providing consistent environments.

Official website: https://www.docker.com/

## Docker architecture

Docker uses a client-server architecture.

Main components:

1. Docker Client: Accepts commands from the user.
2. Docker Daemon (`dockerd`): Manages Docker objects.
3. Docker Engine API: Communication interface.
4. Container Runtime: Runs containers.
5. Docker Registry: Stores images.

### How Docker works

When executing:

`docker run nginx`

The process is broadly:

1. Docker checks whether the image exists locally.
2. If unavailable, it pulls the image from a registry.
3. Docker creates a container.
4. The runtime starts the container.
5. The application runs inside the container.

## Docker commands

### Images

| Command | Purpose |
|---|---|
| `docker pull nginx` | Download an image |
| `docker images` | List local images |
| `docker build -t myapp .` | Build an image |
| `docker rmi IMAGE` | Remove an image |
| `docker tag IMAGE NAME:TAG` | Tag an image |
| `docker push NAME:TAG` | Push image to a registry |

### Containers

| Command | Purpose |
|---|---|
| `docker run nginx` | Create and start a container |
| `docker run -d nginx` | Run in detached mode |
| `docker run -it ubuntu bash` | Interactive terminal |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker start ID` | Start an existing container |
| `docker stop ID` | Stop a container |
| `docker restart ID` | Restart a container |
| `docker rm ID` | Remove a container |
| `docker exec -it ID bash` | Access a running container |
| `docker logs ID` | View container logs |
| `docker inspect ID` | Inspect container details |
| `docker stats` | View resource usage |

### Port mapping

`docker run -d -p 8080:80 nginx`

Format:

`-p HOST_PORT:CONTAINER_PORT`

Here:
- Host port: 8080
- Container port: 80

The application can be accessed through `localhost:8080`.

### Environment variables

`docker run -e NODE_ENV=production myapp`

Environment variables configure applications without hardcoding values into images.

### Container cleanup

`docker container prune`

Removes stopped containers.

`docker image prune`

Removes unused images.

Use cleanup commands carefully.

---

# 3. Dockerfile

## What is a Dockerfile?

A Dockerfile is a text file containing instructions used to build a Docker image.

Example:

```dockerfile
FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "run", "start"]
```

### Dockerfile instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Defines the base image |
| `WORKDIR` | Sets the working directory |
| `COPY` | Copies files into the image |
| `ADD` | Copies files with additional capabilities |
| `RUN` | Executes commands during image building |
| `ENV` | Defines environment variables |
| `ARG` | Defines build-time variables |
| `EXPOSE` | Documents the intended listening port |
| `CMD` | Defines the default startup command |
| `ENTRYPOINT` | Defines the main executable |

### CMD vs ENTRYPOINT

- `CMD`: Provides a default command or arguments that can be overridden.
- `ENTRYPOINT`: Defines the main executable for the container.

They can also be used together.

### Docker image layers

Each relevant Dockerfile instruction creates an image layer.

Layers are cached and reused during subsequent builds.

**Why does this matter?**

Docker build caching reduces unnecessary rebuilding and improves performance.

### Multi-stage builds

Multi-stage builds use multiple `FROM` instructions to separate building from the final runtime image.

Benefits:
- Smaller production images.
- Reduced unnecessary dependencies.
- Better security.
- Cleaner deployment artifacts.

### .dockerignore

Prevents unnecessary files from being copied into the build context.

Example:

```text
node_modules
.next
.git
.env
.env.local
```

Never accidentally include secrets in Docker images.

---

# 4. Docker Networking

Docker networking allows containers to communicate with each other and external systems.

### Networking types

| Network | Purpose |
|---|---|
| Bridge | Default-style isolated container networking |
| Host | Shares the host network namespace |
| None | Disables normal container networking |
| Overlay | Connects containers across multiple Docker hosts |

### Common commands

```bash
docker network ls

docker network create my-network

docker network inspect my-network

docker network connect my-network CONTAINER

docker network disconnect my-network CONTAINER
```

### Container-to-container communication

Containers attached to the same user-defined bridge network can communicate using their container names.

Example:

An API container can connect to a PostgreSQL container using hostname `postgres`.

It does not need to use `localhost`.

**Important:** Inside a container, `localhost` refers to that container itself, not another container or your Mac.

---

# 5. Docker Volumes and Persistent Storage

Containers are generally designed to be replaceable. Their writable container layer is not suitable as the only location for important persistent data.

Docker provides storage mechanisms.

### Types

1. Volumes: Managed by Docker and recommended for persistent application data.
2. Bind mounts: Mount an existing host filesystem path into a container.
3. tmpfs mounts: Store temporary data in memory.

### Commands

```bash
docker volume create postgres_data

docker volume ls

docker volume inspect postgres_data

docker volume rm postgres_data
```

Example:

```bash
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=password \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:17
```

The named volume preserves database data across container replacement.

---

# 6. Docker Compose

## What is Docker Compose?

Docker Compose is a tool used to define and manage multi-container applications using a YAML configuration file.

Instead of starting each container individually, you define services, networks, volumes, and configurations in one file.

Example architecture:

- Next.js frontend.
- Node.js backend.
- PostgreSQL database.
- Redis cache.

### Example compose.yaml

```yaml
services:
  postgres:
    image: postgres:17
    environment:
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: password
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Commands

| Command | Purpose |
|---|---|
| `docker compose up` | Start services |
| `docker compose up -d` | Start in background |
| `docker compose down` | Stop and remove containers/networks |
| `docker compose down -v` | Also remove named volumes |
| `docker compose build` | Build images |
| `docker compose ps` | List services |
| `docker compose logs` | View logs |
| `docker compose exec SERVICE sh` | Execute a command |
| `docker compose restart` | Restart services |

### Why use Compose?

- Simplifies local development.
- Reproducible environments.
- Easier collaboration.
- Centralized configuration.
- Supports service dependencies and health checks.

**Docker vs Docker Compose:**

Docker provides container functionality. Docker Compose coordinates multiple containers through a configuration file.

---

# 7. Container Registries

A container registry stores and distributes container images.

Examples:

- Docker Hub.
- GitHub Container Registry (GHCR).
- Amazon Elastic Container Registry (ECR).
- Google Artifact Registry.

Typical workflow:

1. Build an image.
2. Tag the image.
3. Push it to a registry.
4. Pull it onto another machine.
5. Run the container.

Example:

```bash
docker build -t myapp:v1 .

docker tag myapp:v1 username/myapp:v1

docker push username/myapp:v1
```

---

# 8. Kubernetes

## What is Kubernetes?

Kubernetes is an open-source container orchestration platform.

It automates the deployment, scaling, networking, and management of containerized applications.

It is commonly abbreviated as **K8s**.

Official website: https://kubernetes.io/

### Why do we need Kubernetes?

Imagine running 100 containers across multiple machines.

Manually managing them becomes difficult.

Questions arise:

- What if a container crashes?
- What if a machine fails?
- What if traffic increases?
- How do we scale the application?
- How do we deploy a new version without downtime?

Kubernetes helps automate these operations.

### Docker vs Kubernetes

| Docker | Kubernetes |
|---|---|
| Container platform | Container orchestration platform |
| Builds images | Uses container images |
| Runs containers | Manages workloads across a cluster |
| Docker Compose manages local multi-container apps | Manages cluster workloads |
| Primarily focuses on container lifecycle | Focuses on desired application state |

Kubernetes does not require Docker Engine specifically. It works with compatible container runtimes such as containerd and CRI-O.

---

# 9. Kubernetes Architecture

A Kubernetes cluster consists of:

1. Control Plane.
2. Worker Nodes.

### Control Plane components

| Component | Responsibility |
|---|---|
| API Server | Exposes the Kubernetes API |
| etcd | Stores cluster state and configuration |
| Scheduler | Assigns Pods to suitable nodes |
| Controller Manager | Runs controllers to maintain desired state |
| Cloud Controller Manager | Integrates with supported cloud infrastructure |

### Worker Node components

| Component | Responsibility |
|---|---|
| Kubelet | Ensures Pods are running on the node |
| Kube-proxy | Implements Service networking where applicable |
| Container Runtime | Runs containers |

### What is a cluster?

A cluster is a collection of machines managed by Kubernetes.

### What is a node?

A node is a machine in a Kubernetes cluster.

It can be a physical or virtual machine.

### What is a Pod?

A Pod is the smallest deployable unit in Kubernetes.

It can contain one or more tightly coupled containers that share network and storage resources.

Usually, one main application container runs per Pod.

### What is a Deployment?

A Deployment manages the desired number of replicated Pods.

It supports:
- Scaling.
- Rolling updates.
- Rollbacks.
- Replacement of failed Pods.

### What is a ReplicaSet?

A ReplicaSet ensures that the specified number of Pod replicas exists.

A Deployment manages ReplicaSets.

---

# 10. Kubernetes Objects

| Object | Purpose |
|---|---|
| Pod | Runs application containers |
| Deployment | Manages replicated Pods |
| ReplicaSet | Maintains Pod replicas |
| Service | Provides stable access to Pods |
| Namespace | Logical resource isolation |
| ConfigMap | Stores non-sensitive configuration |
| Secret | Stores sensitive configuration |
| Ingress | Manages external HTTP/HTTPS routing |
| PersistentVolume | Represents storage resources |
| PersistentVolumeClaim | Requests persistent storage |
| StatefulSet | Manages stateful workloads |
| DaemonSet | Runs Pods on selected nodes |
| Job | Runs a task to completion |
| CronJob | Runs scheduled jobs |

### Kubernetes Services

Pods are replaceable, so their IP addresses can change.

Services provide a stable network endpoint.

Types:

- ClusterIP: Internal cluster access.
- NodePort: Exposes a service through a port on each node.
- LoadBalancer: Requests an external load balancer when supported by the environment.
- ExternalName: Maps a Service name to an external DNS name.

### ConfigMap vs Secret

ConfigMap:
- Stores non-sensitive configuration.
- Can be injected as environment variables or mounted as files.

Secret:
- Stores sensitive configuration.
- Can be injected as environment variables or mounted as files.
- Requires proper access control and encryption configuration.

Kubernetes Secrets are not automatically secure merely because they are called Secrets.

---

# 11. Kubernetes YAML

Kubernetes uses declarative configuration.

Example Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-app

spec:
  replicas: 3

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app

    spec:
      containers:
        - name: my-app
          image: nginx:1.27
          ports:
            - containerPort: 80
```

This declares that Kubernetes should maintain three replicas of the application.

### Common YAML fields

- `apiVersion`: API version.
- `kind`: Resource type.
- `metadata`: Name, namespace, labels.
- `spec`: Desired configuration.
- `status`: Current observed state, maintained by Kubernetes.

### Declarative vs Imperative

Imperative:

```bash
kubectl create deployment nginx --image=nginx
```

Declarative:

```bash
kubectl apply -f deployment.yaml
```

Declarative configuration describes the desired state.

Kubernetes controllers continuously work toward that state.

---

# 12. Kubernetes Commands

### Cluster and nodes

```bash
kubectl cluster-info

kubectl get nodes

kubectl describe node NODE_NAME
```

### Pods

```bash
kubectl get pods

kubectl get pods -A

kubectl describe pod POD_NAME

kubectl logs POD_NAME

kubectl exec -it POD_NAME -- sh
```

### Deployments

```bash
kubectl get deployments

kubectl apply -f deployment.yaml

kubectl delete -f deployment.yaml

kubectl scale deployment my-app --replicas=5

kubectl rollout status deployment/my-app

kubectl rollout history deployment/my-app

kubectl rollout undo deployment/my-app
```

### Services

```bash
kubectl get services

kubectl describe service SERVICE_NAME
```

### Debugging

```bash
kubectl get events

kubectl describe pod POD_NAME

kubectl logs POD_NAME

kubectl logs POD_NAME --previous
```

---

# 13. Kubernetes Scaling and Deployment Strategies

### Horizontal scaling

Increasing or decreasing the number of Pod replicas.

Example:

```bash
kubectl scale deployment my-app --replicas=5
```

### Horizontal Pod Autoscaler (HPA)

Automatically adjusts the number of Pod replicas based on configured metrics.

Requires suitable metrics infrastructure.

### Vertical scaling

Increasing or decreasing CPU and memory resources allocated to containers.

### Rolling updates

Gradually replaces old Pods with new Pods.

Helps deploy new application versions without taking the entire application offline.

### Rollback

Reverts a Deployment to an earlier revision.

```bash
kubectl rollout undo deployment/my-app
```

### Health probes

| Probe | Purpose |
|---|---|
| Liveness | Checks whether a container needs restarting |
| Readiness | Checks whether a container should receive traffic |
| Startup | Allows slow-starting applications time to initialize |

### Resource requests and limits

Requests: Resources Kubernetes uses when scheduling a Pod.

Limits: Maximum resource consumption enforced for the container.

---

# 14. Kubernetes Storage

Containers are replaceable, so stateful applications need persistent storage.

Important concepts:

- PersistentVolume (PV).
- PersistentVolumeClaim (PVC).
- StorageClass.
- Volume mounts.

A PVC requests storage, and Kubernetes provisions or binds suitable storage according to the cluster configuration.

Useful for:
- Databases.
- Stateful applications.
- Persistent application data.

Stateful applications may use StatefulSets when stable identities and storage associations are required.

---

# 15. Advanced Container & Kubernetes Concepts

| Topic | Purpose |
|---|---|
| Helm | Kubernetes package manager |
| Ingress Controller | Implements Ingress routing |
| RBAC | Controls access to Kubernetes resources |
| NetworkPolicy | Controls Pod network communication |
| Service Discovery | Finds services dynamically |
| Namespaces | Separates cluster resources |
| Container Security | Secures images and runtime |
| Image Scanning | Detects known vulnerabilities |
| Prometheus | Metrics collection |
| Grafana | Monitoring dashboards |
| Centralized Logging | Aggregates application logs |
| CI/CD | Automates build, test, and deployment |
| Terraform | Infrastructure as Code |

### Important tools

- Minikube: Local Kubernetes cluster.
- kind: Kubernetes clusters using Docker containers as nodes.
- kubectl: Kubernetes command-line tool.
- Helm: Package manager for Kubernetes.
- Prometheus: Monitoring and metrics.
- Grafana: Metrics visualization.
- Terraform: Infrastructure provisioning.

---

# 16. Practical Development Workflow

A typical containerized development and deployment workflow:

1. Write application code.
2. Build a Docker image.
3. Test the container locally.
4. Define services using Docker Compose.
5. Run local databases and dependencies.
6. Push the image to a container registry.
7. Configure Kubernetes manifests.
8. Deploy the application.
9. Expose it through a Service and, where needed, Ingress.
10. Monitor logs, metrics, and resource usage.

### Example full-stack architecture

```text
                 USERS
                   |
             Load Balancer
                   |
                Ingress
                   |
             Kubernetes
                   |
          -------------------
          |                 |
      Frontend Pod       Backend Pods
      (Next.js)          (Node.js)
                             |
                   -------------------
                   |                 |
               PostgreSQL          Redis
```

The databases can be hosted inside Kubernetes, outside the cluster, or through managed cloud services.

For many production applications, managed databases are a sensible choice.

---

# 17. Docker vs Kubernetes vs Docker Compose

| Feature | Docker | Docker Compose | Kubernetes |
|---|---|---|---|
| Build images | Yes | Uses Docker build capabilities | No, uses existing images |
| Run containers | Yes | Yes | Yes, through container runtimes |
| Multiple services | Manually or through APIs | Yes | Yes |
| Service networking | Yes | Yes | Yes |
| Persistent volumes | Yes | Yes | Yes |
| Automated orchestration | Limited | Local multi-container lifecycle | Advanced cluster orchestration |
| Automatic scaling | No general cluster autoscaling | No built-in cluster autoscaling | Yes, when configured |
| Rolling deployments | Not a general orchestration feature | Limited | Yes |
| Production orchestration | Depends on deployment model | Not a Kubernetes replacement | Yes |

---

# 18. Common Interview Questions

### Containerization

1. What is containerization?
2. What is the difference between a container and a virtual machine?
3. What is a Docker image?
4. What is a container?
5. What is a container registry?
6. How does Docker work internally?
7. What are namespaces and cgroups?

### Docker

8. What is a Dockerfile?
9. Difference between CMD and ENTRYPOINT?
10. Difference between COPY and ADD?
11. What are Docker image layers?
12. What is Docker build caching?
13. What is a multi-stage build?
14. What is `.dockerignore`?
15. What is the difference between a volume and bind mount?
16. How does Docker networking work?
17. What is port mapping?
18. What is Docker Compose?
19. How do containers communicate with one another?
20. How do you persist database data in Docker?

### Kubernetes

21. What is Kubernetes and why do we need it?
22. Explain Kubernetes architecture.
23. What is a Pod?
24. What is a Node?
25. What is a Cluster?
26. What is a Deployment?
27. What is a ReplicaSet?
28. What is the difference between a Pod and a Deployment?
29. What are Kubernetes Services?
30. Explain ClusterIP, NodePort, and LoadBalancer.
31. What are ConfigMaps and Secrets?
32. What is Ingress?
33. What is a PersistentVolume and PVC?
34. What is a StatefulSet?
35. What happens when a Pod crashes?
36. What is HPA?
37. Explain rolling updates and rollbacks.
38. What are liveness, readiness, and startup probes?
39. What is Helm?
40. How do you debug a failing Kubernetes Pod?

### Scenario-based questions

41. Your container works locally but fails on another machine. How would you debug it?
42. A Docker container exits immediately. What would you investigate?
43. Your database loses data after a container restart. Why?
44. Your application cannot connect to PostgreSQL in another container. How would you debug networking?
45. A Kubernetes Pod is stuck in `Pending`. What could be the reason?
46. A Pod is stuck in `CrashLoopBackOff`. How would you investigate?
47. Your application receives traffic but returns errors. How would you debug its Service and readiness configuration?
48. How would you deploy a new application version without interrupting users?
49. How would you scale an application during increased traffic?
50. How would you secure sensitive environment variables in Kubernetes?

---

# 19. Quick Revision Cheat Sheet

| Term | One-line definition |
|---|---|
| Containerization | Packaging applications into isolated environments |
| Docker | Platform for building and running containers |
| Image | Template used to create containers |
| Container | Running instance of an image |
| Dockerfile | Instructions for building images |
| Docker Compose | Tool for managing multi-container applications |
| Docker Volume | Persistent storage managed by Docker |
| Registry | Image storage and distribution system |
| Kubernetes | Container orchestration platform |
| Cluster | Collection of machines managed by Kubernetes |
| Node | Machine in a Kubernetes cluster |
| Pod | Smallest deployable Kubernetes unit |
| Deployment | Manages replicated application Pods |
| ReplicaSet | Maintains desired Pod replicas |
| Service | Stable network endpoint for Pods |
| Ingress | HTTP/HTTPS routing into a cluster |
| ConfigMap | Non-sensitive application configuration |
| Secret | Sensitive configuration data |
| HPA | Automatically scales Pod replicas |
| Helm | Kubernetes package manager |
| kubectl | Kubernetes CLI |

---

# 20. Official Learning Resources

### Docker
- Getting Started: https://docs.docker.com/get-started/
- Dockerfile: https://docs.docker.com/reference/dockerfile/
- Docker Compose: https://docs.docker.com/compose/
- Docker Networking: https://docs.docker.com/engine/network/
- Docker Storage: https://docs.docker.com/engine/storage/

### Kubernetes
- Concepts: https://kubernetes.io/docs/concepts/
- Interactive Basics: https://kubernetes.io/docs/tutorials/kubernetes-basics/
- Workloads: https://kubernetes.io/docs/concepts/workloads/
- Services and Networking: https://kubernetes.io/docs/concepts/services-networking/
- Storage: https://kubernetes.io/docs/concepts/storage/
- kubectl: https://kubernetes.io/docs/reference/kubectl/

### Additional tools
- Minikube: https://minikube.sigs.k8s.io/
- kind: https://kind.sigs.k8s.io/
- Helm: https://helm.sh/
- Prometheus: https://prometheus.io/
- Grafana: https://grafana.com/
- Terraform: https://developer.hashicorp.com/terraform

---

## Final takeaway

**Docker answers:** How do I package and run my application consistently?

**Docker Compose answers:** How do I run multiple services together during development?

**Kubernetes answers:** How do I orchestrate, scale, monitor, and maintain containerized applications across a cluster?

These technologies are fundamental building blocks of modern cloud-native and backend engineering.
