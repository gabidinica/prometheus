# Demo Project: Install Prometheus Stack in Kubernetes

## Technologies Used
- Prometheus  
- Kubernetes  
- Helm  
- AWS EKS  
- eksctl  
- Grafana  
- Linux  

## Project Description
- Setup an **EKS cluster** using `eksctl`.  
- Deploy **Prometheus**, **Alertmanager**, and **Grafana** in the cluster as part of the **Prometheus Operator** using a Helm chart.  

---

We’re gonna use the project with microservices from here:  
[Microservices Project](https://github.com/gabidinica/container-orchestration-with-k8/tree/main/microservices)  

Clone this project and we’re gonna use the `config.yaml` file.

---

## Create EKS Cluster using eksctl

### 1. Create Cluster
In the terminal, run:
```bash
eksctl create cluster
```

> Creates a cluster in the default AWS region, uses default AWS credentials and spins up 2 worker nodes

### 2. Verify nodes:
`kubectl get nodes`

### 3. Apply the config.yaml file to deploy the microservices application:
`kubectl apply -f config.yaml`

### 4. Use the following command to verify that the applications are running:
```bash
kubectl get pods
```

> Helps ensure that the microservices and other components are up and running.

## Deploy Prometheus Stack using Helm

### 1. Add Helm Repository
Add the repository where the Prometheus Helm chart is located:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

### 2. Update Helm Repositories:
```bash
heml repo update
```

## Install Prometheus into Its Own Namespace

### 1. Create Monitoring Namespace
```bash
kubectl create namespace monitoring
```

### 2. Install Prometheus Stack:
```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

### 3. Verify Prometheus Deployment:
```bash
kubectl get all -n monitoring
```

### 4. Check All Prometheus Stack Pods
```bash
kubectl get all -n monitoring
```

### 5. Check Custom Resource Definitions (CRDs):
```bash
kubectl get crd -n monitoring
```

> CRDs extend the Kubernetes API with custom resources.

### 6. Check StatefulSets
```bash
kubectl get statefulset -n monitoring
```

### 7. Describe Prometheus StatefulSet:
```bash
kubectl describe statefulset prometheus-monitoring-kube-prometheus-prometheus -n monitoring > prom.yaml
```

> Saves the detailed StatefulSet information to prom.yaml.

### 8. Check Deployments:
```bash
kubectl get deployment -n monitoring
```

### 9. Describe Prometheus Operator Deployment:
```bash
kubectl describe deployment monitoring-kube-prometheus-operator -n monitoring > oper.yaml
```

> Saves the detailed Deployment information to oper.yaml

## Access Prometheus UI

### 1. Port-Forward Prometheus Service
```bash
kubectl port-forward service/monitoring-kube-prometheus-prometheus -n monitoring 9090:9090 &
```

### 2. Open Prometheus UI

- Copy the IP address `http://localhost:9090` into your browser
- Navigate to Status → Targets to see the list of monitored components

## Access Grafana UI

### 1. Port-Forward Grafana Service
```bash
kubectl port-forward service/monitoring-grafana -n monitoring 8080:8080 &
```

### 2. Open Grafana in Browser

- Copy the IP address:8080 in your browser
- Default credentials:
	- Username: admin
	- Password: prom-operator

## Trigger CPU Spike with Multiple Requests

### 1. Deploy a BusyBox Pod for Curl
Use the following command to run a temporary pod with curl installed:
```bash
kubectl run curl-test --image=radial/busyboxplus:curl -i --tty --rm
```

> This pod can be used to send multiple requests to your application to simulate CPU load.

## Create a Script to Stress Test the Application Endpoint

### 1. Create `test.sh` Script
```bash
vim test.sh
```

### 2. Add the Following Content

```bash
for i in $(seq 1 10000)
do
  curl ae4aee0715edc46b988c6ce67121bf57-1459479566.eu-west-3.elb.amazonaws.com > test.txt
done
```

> This script sends 10,000 requests to the application endpoint (replace the URL with your actual external load balancer endpoint). Output of each request is saved to test.txt.

You can check the changes in Grafana Dashboard.
