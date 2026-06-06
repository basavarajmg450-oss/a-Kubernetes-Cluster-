# 🚀 Kubernetes Hands-on Internship Task Guide

Welcome to the Kubernetes Hands-on Deployment and Management guide. This repository contains the configuration manifests and complete step-by-step instructions to set up a local Kubernetes cluster using **Minikube**, deploy a containerized Nginx web server serving custom content via **ConfigMaps**, expose the application with a **NodePort Service**, and demonstrate manual scaling and self-healing.

This guide is structured to serve as both an execution guide for interns and a submission-ready repository for evaluation.

---

## 📂 Repository Contents

*   [`configmap.yaml`](file:///c:/task%205/configmap.yaml) - Kubernetes ConfigMap storing the custom HTML template.
*   [`deployment.yaml`](file:///c:/task%205/deployment.yaml) - Kubernetes Deployment manifest specifying 2 replicas of Nginx with resource constraints, probes, and volume mounts.
*   [`service.yaml`](file:///c:/task%205/service.yaml) - Kubernetes Service manifest of type `NodePort` mapping port 80 of our application to port 32000 of the cluster nodes.

---

## 🛠️ Step 1: Local Environment Installation & Setup

To complete this task, you need to install **Docker** (or another container runtime supported by Minikube), **Minikube** (local cluster coordinator), and **kubectl** (the Kubernetes command-line interface).

### 🖥️ Windows Setup
1.  **Docker Desktop**: Download and run the installer from the [Docker Desktop Official Site](https://www.docker.com/products/docker-desktop/). Ensure WSL 2 backend integration is enabled.
2.  **Minikube**: Open PowerShell as Administrator and run:
    ```powershell
    New-Item -Path 'C:\minikube' -ItemType Directory -Force
    Invoke-WebRequest -Uri "https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe" -OutFile "C:\minikube\minikube.exe"
    [Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\minikube", "Machine")
    ```
3.  **kubectl**: Open PowerShell as Administrator and run:
    ```powershell
    Invoke-WebRequest -Uri "https://dl.k8s.io/release/v1.30.0/bin/windows/amd64/kubectl.exe" -OutFile "C:\minikube\kubectl.exe"
    ```

### 🍎 macOS Setup
Install the dependencies using Homebrew:
```bash
# Install Docker (or Colima)
brew install --cask docker

# Install Minikube
brew install minikube

# Install kubectl
brew install kubernetes-cli
```

### 🐧 Linux Setup (Ubuntu/Debian)
```bash
# Update package index and install Docker
sudo apt-get update
sudo apt-get install -y docker.io
sudo usermod -aG docker $USER && newgrp docker

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

---

## 🚀 Step 2: Spin Up the Cluster

Once installed, spin up your local single-node cluster.

1.  **Start Minikube** (Specify docker as driver explicitly if needed):
    ```bash
    minikube start --driver=docker
    ```
2.  **Verify Cluster Status**:
    ```bash
    kubectl cluster-info
    kubectl get nodes
    ```
    *Expectation:* You should see one node named `minikube` in the `Ready` status.

---

## 📦 Step 3: Deploy the Application

Now we will deploy the ConfigMap, the Deployment, and the Service in sequence. 

### 1. Apply Manifests
Apply the manifests to the cluster in the following logical order (configuration -> workloads -> networking):
```bash
# 1. Apply the ConfigMap
kubectl apply -f configmap.yaml

# 2. Apply the Deployment Workload
kubectl apply -f deployment.yaml

# 3. Apply the Service Expose
kubectl apply -f service.yaml
```

### 2. Verify Workload Creation
Check that your workloads are provisioned successfully:
```bash
# Check the deployment status
kubectl get deployments

# Check if the pods are in Running status and Ready (1/1)
kubectl get pods -l app=web-server

# Check the service creation and exposed ports
kubectl get services web-server-service
```

### 3. Retrieve Web Server Access URL
Because Minikube runs in an isolated network/container environment, you must use Minikube's built-in command to retrieve the direct access URL:
```bash
# Print the direct access URL
minikube service web-server-service --url
```
Copy the printed URL (e.g., `http://192.168.49.2:32000` or `http://127.0.0.1:<random_port>`) and open it in a web browser. You should see the custom blue/purple **"Kubernetes Deployment Successful!"** web page.

---

## 📈 Step 4: Scale the Workload

One of Kubernetes' core strengths is its ease of scaling. Let's scale our deployment from **2 to 5 replicas**.

1.  **Run the Scale Command**:
    ```bash
    kubectl scale deployment/web-server-deployment --replicas=5
    ```
2.  **Verify the Scaling Progression**:
    ```bash
    kubectl get pods -w
    ```
    *Observation:* Notice the rapid creation of 3 new pods. Press `Ctrl+C` to exit the watch screen.
3.  **Confirm 5 Pods are Running**:
    ```bash
    kubectl get pods -l app=web-server
    ```

---

## 📸 Step 5: Capture Submission Screenshots

To prove completion of the internship assignment, capture and document screenshots of the following commands:

1.  **Running Pods**: Shows that all scaled replicas are functional.
    ```bash
    kubectl get pods -o wide
    ```
    *(Capture screenshot highlighting the 5 running pods)*
    
2.  **Service Details**: Verifies IP allocation and NodePort configuration.
    ```bash
    kubectl get service web-server-service
    ```
    *(Capture screenshot showcasing the NodePort mappings)*

3.  **Pod Specifications and Health Probes**:
    Identify one of your pod names (e.g., `web-server-deployment-xxxxxxxxx-xxxx`) and run:
    ```bash
    kubectl describe pod <pod-name>
    ```
    *(Capture screenshot highlighting the 'Liveness' and 'Readiness' probe status)*

4.  **Live Logs**: Check the standard output logs of one of your pods to see browser requests hitting the Nginx web server:
    ```bash
    kubectl logs <pod-name>
    ```
    *(Capture screenshot showcasing HTTP requests hitting the endpoint)*

---

## 💡 Troubleshooting Common Pitfalls

*   **Error: `ImagePullBackOff` or `ErrImagePull`**
    *   *Cause:* The container image cannot be downloaded.
    *   *Fix:* Check internet connectivity, or ensure the image name (e.g., `nginx:alpine`) is spelled correctly.
*   **Error: Pods stuck in `Pending`**
    *   *Cause:* Insufficient resource limits or Node capacity.
    *   *Fix:* Since Minikube runs on your local system, ensure your Minikube has sufficient CPU/Memory. Restart Minikube using `minikube delete` and run `minikube start --memory=4096 --cpus=3` to allocate more resources.
*   **Error: Cannot connect to NodePort URL**
    *   *Cause:* Docker network isolation driver (common on macOS/Windows WSL2).
    *   *Fix:* Use `minikube service web-server-service` without the `--url` flag to let Minikube set up a tunnel proxy directly to your browser automatically.

---

## 📖 Deep-Dive: Core Kubernetes Concepts

This implementation demonstrates several key building blocks of container orchestration:

*   **Declarative Configuration**: Rather than using manual CLI run commands, we declared our system state in YAML files. Kubernetes constantly works to align actual state with this declared configuration.
*   **Decoupled Architecture**: By binding the Custom HTML content inside a `ConfigMap` and mounting it as a volume, our application code remains isolated from configuration details.
*   **Self-Healing**: If a Pod container crashes or a Readiness Probe fails, Kubernetes automatically restarts the container or replaces the unhealthy Pod.
*   **Service Load Balancing**: The `Service` component distributes traffic across all active pod endpoints, managing DNS and networking abstractly.

---

## 🎓 DevOps Interview Q&A

Here are structural, industry-aligned answers to the 8 required interview questions:

### 1. What is Kubernetes?
**Kubernetes (K8s)** is an open-source container orchestration platform designed to automate deploying, scaling, and managing containerized applications. It eliminates the manual work of running containers on host machines by managing load balancing, self-healing (restarting failed containers), automated rollouts/rollbacks, and configuration/storage provisioning across a cluster of server nodes.

### 2. What is the role of the `kubelet`?
The `kubelet` is the primary agent running on **every node** in a Kubernetes cluster. Its role is to ensure containers described in PodSpecs are running and healthy. Specifically, it:
*   Receives pod instructions (PodSpecs) from the API server.
*   Interacts with the local container runtime (like Docker or containerd) to pull images and run containers.
*   Monitors container health and reports state, resources, and events back to the Kubernetes control plane.

### 3. Explain Pods, Deployments, and Services.
*   **Pod**: The smallest deployable unit in Kubernetes. It represents a single instance of a running process in the cluster and can hold one or more tightly-coupled containers sharing the same network namespace, storage volumes, and IP address.
*   **Deployment**: A higher-level controller that manages declarative updates to Pods. It defines the desired state of Pods (e.g., image version, replica count) and handles rollouts, rollbacks, and self-healing of those Pods.
*   **Service**: An abstraction layer that defines a logical set of Pods and a policy to access them. Since Pods are ephemeral and their IP addresses change constantly when rescheduled, a Service provides a stable, persistent DNS name and IP address to access Pods, load-balancing traffic among them.

### 4. How do you scale in Kubernetes?
Kubernetes supports scaling at multiple layers:
*   **Manual Scaling**: Resizing workloads using commands like `kubectl scale deployment/<deployment-name> --replicas=<count>` or editing the manifest file directly.
*   **Horizontal Pod Autoscaler (HPA)**: Automatically adjusts the number of pods based on resource utilization metrics (e.g., target CPU utilization of 80%) using the Metrics Server.
*   **Vertical Pod Autoscaler (VPA)**: Adjusts the CPU and memory resource requests/limits of existing containers over time.
*   **Cluster Autoscaler**: Automatically provisions additional physical/virtual nodes for the cluster when resources are depleted.

### 5. What is a Namespace?
A **Namespace** is a logical partition within a Kubernetes cluster. It allows you to isolate resources (such as Pods, Deployments, and Services) belonging to different environments (e.g., `dev`, `staging`, `prod`), projects, or teams inside the same physical cluster. Namespaces provide scope for resource naming, network access policies, and resource quotas/limits.

### 6. What is the difference between `ClusterIP`, `NodePort`, and `LoadBalancer` services?
*   **ClusterIP** (Default): Exposes the Service on a cluster-internal IP. This service is *only* reachable from within the Kubernetes cluster. Perfect for internal databases or backend services.
*   **NodePort**: Exposes the Service on each cluster node's IP at a static port (in the 30000-32767 range). External clients can access the service by hitting `<NodeIP>:<NodePort>`.
*   **LoadBalancer**: Integrates with cloud provider load balancers (AWS ELB, GCP Load Balancer) to provision an external public IP addresses that forwards traffic directly to node ports. Useful for production-facing applications.

### 7. What are ConfigMaps?
A **ConfigMap** is an API object used to store non-confidential data in key-value pairs. By separating configuration artifacts (such as environment variables, config files, command-line arguments) from container images, ConfigMaps allow you to deploy the exact same container image across different environments (dev, test, production) without rebuilding the container.

### 8. How do you perform rolling updates?
A **Rolling Update** is the default deployment strategy in Kubernetes. It updates pod instances incrementally (one by one or in small batches) to avoid application downtime. 
*   **Process**: Kubernetes creates a new ReplicaSet alongside the old one. It spins up new pods (using the updated image) and slowly terminates old pods as new ones pass their readiness checks.
*   **Control Parameters**: Controlled in the Deployment spec under `strategy.rollingUpdate` using:
    *   `maxSurge`: Maximum number of pods that can be created above the desired replica count during update.
    *   `maxUnavailable`: Maximum number of pods that can be unavailable during update.
