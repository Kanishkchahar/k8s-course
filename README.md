# SynergyChat — Kubernetes Course (Boot.dev)

This repository contains Kubernetes manifests, configuration files, and setup instructions for deploying the **SynergyChat** microservices application as part of the [Boot.dev](https://www.boot.dev) Kubernetes course.

---

## Architecture Overview

The **SynergyChat** application consists of multiple microservices running on a local Kubernetes cluster (Minikube):

* **`synergychat-web`**: Web frontend serving the user interface on port `8080`.
* **`synergychat-api`**: Backend API service configured through environment variables supplied by a `ConfigMap`.
* **`synergychat-crawler`**: Background crawler worker service that ingests and processes keyword streams.
* **`nginx` (`simple-pod.yaml`)**: Introductory standalone pod used for exploring basic pod lifecycle and networking.

---

## Repository Structure & Manifests

| Component | Manifest | Kind | Description |
|---|---|---|---|
| **API Config** | [`api-configmap.yaml`](file:///home/kanishk/Desktop/k8s-course/api-configmap.yaml) | `ConfigMap` | Sets `API_PORT: "8080"` |
| **API Deployment** | [`api-deployment.yaml`](file:///home/kanishk/Desktop/k8s-course/api-deployment.yaml) | `Deployment` | Runs `bootdotdev/synergychat-api:latest` (1 replica) |
| **API Service** | [`api-service.yaml`](file:///home/kanishk/Desktop/k8s-course/api-service.yaml) | `Service` | Exposes API internally (`ClusterIP`) on port `80` &rarr; `8080` |
| **Crawler Config** | [`crawler-configmap.yaml`](file:///home/kanishk/Desktop/k8s-course/crawler-configmap.yaml) | `ConfigMap` | Sets crawler port and sentiment keywords |
| **Crawler Deployment** | [`crawler-deployment.yaml`](file:///home/kanishk/Desktop/k8s-course/crawler-deployment.yaml) | `Deployment` | Runs `bootdotdev/synergychat-crawler:latest` with `envFrom` |
| **Crawler Service** | [`crawler-service.yaml`](file:///home/kanishk/Desktop/k8s-course/crawler-service.yaml) | `Service` | Internal service routing traffic to crawler/web pods |
| **Web Deployment** | [`web-deployment.yaml`](file:///home/kanishk/Desktop/k8s-course/web-deployment.yaml) | `Deployment` | Runs `bootdotdev/synergychat-web:latest` (1 replica) |
| **Web Service** | [`web-service.yaml`](file:///home/kanishk/Desktop/k8s-course/web-service.yaml) | `Service` | Exposes frontend on port `80` &rarr; `8080` |
| **Simple Pod** | [`simple-pod.yaml`](file:///home/kanishk/Desktop/k8s-course/simple-pod.yaml) | `Pod` | Single Nginx container pod (`nginx:1.14.2`) |

---

## Prerequisites & Installation (Arch Linux)

The course uses **Minikube** with the **Docker** driver.

```bash
# 1. Install Docker, kubectl, and Minikube
sudo pacman -S docker kubectl minikube

# 2. Start and enable Docker service
sudo systemctl enable --now docker

# 3. Add your user to the docker group
sudo usermod -aG docker $USER
newgrp docker
```

> **Note**: Log out and back in if `newgrp docker` does not pick up permissions immediately.

### Verify Installation

```bash
docker run hello-world
kubectl version --client
minikube version
```

---

## Starting Minikube

Start Minikube with the CORS flag required by the Boot.dev testing suite:

```bash
minikube start --extra-config="apiserver.cors-allowed-origins=['http://boot.dev']"
```

- If Minikube doesn't auto-detect the Docker driver, append `--driver=docker`.
- Confirm cluster availability:

```bash
kubectl get nodes
```
You should see `minikube` in status `Ready`. Keep Minikube running while completing course lessons.

---

## Deployment & Usage

### 1. Apply ConfigMaps

```bash
kubectl apply -f api-configmap.yaml
kubectl apply -f crawler-configmap.yaml
```

### 2. Apply Deployments

```bash
kubectl apply -f api-deployment.yaml
kubectl apply -f crawler-deployment.yaml
kubectl apply -f web-deployment.yaml
```

### 3. Expose with Services

```bash
kubectl apply -f api-service.yaml
kubectl apply -f crawler-service.yaml
kubectl apply -f web-service.yaml
```

### 4. Verify Workloads

```bash
# Check running pods
kubectl get pods

# Check active services
kubectl get svc
```

### 5. Accessing Services Locally

To access the web or API service in your browser during local development:

```bash
# Forward port from service to localhost:
kubectl port-forward svc/web-service 8080:80

# Or let Minikube launch/tunnel the service:
minikube service web-service
```

---

## Useful Commands & Cheatsheet

### Minikube Commands

| Command | Description |
|---|---|
| `minikube status` | Check whether the local cluster is running |
| `minikube stop` | Pause cluster and preserve current state |
| `minikube start` | Resume cluster (CORS flag not required after first run) |
| `minikube delete` | Teardown the cluster and wipe state |
| `minikube dashboard` | Open Kubernetes web dashboard |

### Common `kubectl` Operations

```bash
# Inspect resources
kubectl get pods -o wide
kubectl get deployments
kubectl get services

# Debugging and logs
kubectl describe pod <pod-name>
kubectl logs -f <pod-name>
kubectl exec -it <pod-name> -- sh

# Apply and delete
kubectl apply -f <manifest.yaml>
kubectl delete -f <manifest.yaml>
```

---

## Quality of Life / Shell Configuration

Add bash / zsh auto-completion and the standard `k` alias to `~/.bashrc` (or `~/.zshrc`):

```bash
source <(kubectl completion bash)
alias k=kubectl
complete -o default -F __start_kubectl k
```

Helpful auxiliary tools:

```bash
sudo pacman -S k9s helm
```

---

## Troubleshooting

| Problem | Resolution |
|---|---|
| `permission denied` on `/var/run/docker.sock` | Run `sudo usermod -aG docker $USER`, log out, and log back in. |
| `minikube start` fails | Ensure Docker daemon is active: `systemctl status docker`. |
| Dirty or conflicting cluster state | Run `minikube stop && minikube delete`, then start fresh. |
| Configuration changes not reflected | Run `minikube delete` and start again with updated flags. |
| Leftover Kind / other clusters conflicting | Remove dangling Kind clusters: `kind delete cluster --name learn`. |
| Wrong `kubectl` context active | Switch back to Minikube: `kubectl config use-context minikube`. |

> **Warning**: Avoid running `minikube` with `sudo`. Always manage permissions via the `docker` user group.
