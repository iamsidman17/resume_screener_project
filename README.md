Cloud Computing Assignment 1: Auto-Scaling Demonstration (Go Version)Roll Number: da24c019Student: Sidharth ManakilOverviewThis project implements a high-performance client-server application in Go to demonstrate and compare two container orchestration strategies on a local machine:A fixed-size deployment (3 replicas) using Docker Swarm.A dynamically auto-scaling deployment (3 to 10 replicas) using Kubernetes and a Horizontal Pod Autoscaler (HPA).The server performs a CPU-intensive task to generate a measurable load, and a concurrent client is used to test the performance and behavior of each setup.PrerequisitesTo run this project, you will need:Go (v1.20+)Docker Desktop (with Swarm and Kubernetes enabled)kubectl configured on your machineIncluded Filesserver.go: The CPU-intensive web server.client.go: The concurrent, rate-based load-testing client.Dockerfile: Multi-stage Dockerfile for building the server.deployment.yaml, service.yaml, hpa.yaml: Kubernetes manifests.da24c019... .txt files: The four required output files from the experiments.Final_Report.md: The detailed analysis of the project.How to RunStep 1: Build the Multi-Platform Docker ImageThis command builds the Docker image for both local (ARM64) and cloud (AMD64) environments and pushes it to Docker Hub. This only needs to be run once.docker buildx build --platform linux/amd64,linux/arm64 -t iamsidman17/reverser-app --push .
Experiment A: Docker Swarm Test (Fixed Replicas)Initialize Docker Swarm (if not already active):This command turns your Docker engine into a Swarm manager. You only need to run this once.docker swarm init
Start the service:docker service create --name reverser-service --replicas 3 -p 8080:8080 iamsidman17/reverser-app
Run the tests:# Low-load test (10 RPS for 60s)
go run client.go --platform dockerswarm --rate 10 --duration 60 --roll_number da24c019 --host http://localhost:8080

# High-load test (5000 RPS for 60s)
go run client.go --platform dockerswarm --rate 5000 --duration 60 --roll_number da24c019 --host http://localhost:8080
Clean up:docker service rm reverser-service
# Optional: To disable Swarm mode completely
docker swarm leave --force
Experiment B: Kubernetes Test (Auto-Scaling)Enable Kubernetes:In Docker Desktop Settings, go to the Kubernetes tab and ensure the "Enable Kubernetes" box is checked.Set kubectl context to your local cluster:kubectl config use-context docker-desktop
Install the Metrics Server (if not already installed):kubectl apply -f [https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml](https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
Deploy the application:kubectl apply -f .
Run the tests:In a separate terminal, you can monitor the auto-scaling with kubectl get hpa -w.# Low-load test (10 RPS for 60s)
go run client.go --platform kubernetes --rate 10 --duration 60 --roll_number da24c019 --host http://localhost

# High-load test (5000 RPS for 60s)
go run client.go --platform kubernetes --rate 5000 --duration 60 --roll_number da24c019 --host http://localhost
Clean up:kubectl delete -f .
Output FilesThe tests will generate four files with the results as required by the assignment.Note: The client is programmed to name the high-load test file with 10000 to match the original assignment's specific filename requirement, even when running at a different rate like 5000 RPS.da24c019dockerswarm10.txtda24c019dockerswarm10000.txtda24c019kubernetes10.txtda24c019kubernetes10000.txt
