# Cloud Computing Assignment 1: Auto-Scaling Demonstration (Go Version)

**Roll Number:** da24c019  
**Student:** Sidharth Manakil  

---

## 📖 Overview
This project implements a **high-performance client-server application in Go** to demonstrate and compare two container orchestration strategies on a local machine:

1. **Docker Swarm (Fixed-size deployment)** – 3 replicas.  
2. **Kubernetes (Auto-scaling deployment)** – 3 to 10 replicas using a **Horizontal Pod Autoscaler (HPA)**.

The server performs a **CPU-intensive task** to generate a measurable load, and a concurrent client is used to test the **performance and behavior** of each setup.

---

## 🔧 Prerequisites
To run this project, you will need:
- **Go** (v1.20+)
- **Docker Desktop** (with **Swarm** and **Kubernetes** enabled)
- **kubectl** configured on your machine

---

## 📂 Included Files
- `server.go` – The CPU-intensive web server.
- `client.go` – The concurrent, rate-based load-testing client.
- `Dockerfile` – Multi-stage Dockerfile for building the server.
- `deployment.yaml`, `service.yaml`, `hpa.yaml` – Kubernetes manifests.
- `da24c019... .txt` – Four required output files from the experiments.
- `Final_Report.md` – Detailed analysis of the project.

---

## 🚀 How to Run

### Step 1: Build the Multi-Platform Docker Image
This command builds the Docker image for both **local (ARM64)** and **cloud (AMD64)** environments and pushes it to Docker Hub.  
*(This only needs to be run once.)*
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t iamsidman17/reverser-app --push .
```

---

## ⚡ Experiment A: Docker Swarm Test (Fixed Replicas)

1. **Initialize Docker Swarm**  
   *(If not already active, run this only once.)*
   ```bash
   docker swarm init
   ```

2. **Start the Service**
   ```bash
   docker service create --name reverser-service --replicas 3 -p 8080:8080 iamsidman17/reverser-app
   ```

3. **Run the Tests**
   ```bash
   # Low-load test (10 RPS for 60s)
   go run client.go --platform dockerswarm --rate 10 --duration 60 --roll_number da24c019 --host http://localhost:8080

   # High-load test (5000 RPS for 60s)
   go run client.go --platform dockerswarm --rate 5000 --duration 60 --roll_number da24c019 --host http://localhost:8080
   ```

4. **Clean Up**
   ```bash
   docker service rm reverser-service

   # Optional: To disable Swarm mode completely
   docker swarm leave --force
   ```

---

## ☸️ Experiment B: Kubernetes Test (Auto-Scaling)

1. **Enable Kubernetes**  
   In Docker Desktop Settings, go to the **Kubernetes** tab and ensure the **Enable Kubernetes** box is checked.

2. **Set kubectl Context to Your Local Cluster**
   ```bash
   kubectl config use-context docker-desktop
   ```

3. **Install the Metrics Server (if not already installed)**
   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   kubectl patch deployment metrics-server -n kube-system --type='json'      -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
   ```

4. **Deploy the Application**
   ```bash
   kubectl apply -f .
   ```

5. **Run the Tests**  
   *(In a separate terminal, you can monitor auto-scaling with `kubectl get hpa -w`.)*
   ```bash
   # Low-load test (10 RPS for 60s)
   go run client.go --platform kubernetes --rate 10 --duration 60 --roll_number da24c019 --host http://localhost

   # High-load test (5000 RPS for 60s)
   go run client.go --platform kubernetes --rate 5000 --duration 60 --roll_number da24c019 --host http://localhost
   ```

6. **Clean Up**
   ```bash
   kubectl delete -f .
   ```

---

## 📊 Output Files
The tests will generate the following four files as required by the assignment:

- `da24c019dockerswarm10.txt`  
- `da24c019dockerswarm10000.txt`  
- `da24c019kubernetes10.txt`  
- `da24c019kubernetes10000.txt`  

> **Note:** The client is programmed to name the high-load test file with **10000** to match the original assignment’s filename requirement, even when running at a different rate like **5000 RPS**.

---

## ✅ Summary
This project provides a practical demonstration of **auto-scaling** using Kubernetes compared to a **fixed-replica deployment** in Docker Swarm, highlighting the dynamic scaling capabilities of Kubernetes when subjected to high CPU loads.

---
