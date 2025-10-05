# Demo Project: Configure Monitoring for a Third-Party Application

## Technologies Used
- Prometheus  
- Kubernetes  
- Redis  
- Helm  
- Grafana  

## Project Description
- Monitor **Redis** by using the **Prometheus Exporter**  
- Deploy **Redis service** in the Kubernetes cluster  
- Deploy **Redis Exporter** using a Helm chart  
- Configure **Alert Rules** to notify when:
  - Redis is down  
  - Redis has too many connections  
- Import a **Grafana Dashboard for Redis** to visualize monitoring data in Grafana  

---

## Deploy Redis Exporter

### 1. Overview
- **Prometheus Exporters** are apps that connect to a service and expose metrics for Prometheus.  
- We will use **Redis Exporter** to monitor Redis:  
[Redis Exporter GitHub](https://github.com/oliver006/redis_exporter)  

### 2. Helm Chart
- Use the Prometheus community Helm chart for Redis Exporter:  
[Prometheus Redis Exporter Helm Chart](https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus-redis-exporter)  

### 3. Custom Values
- The project includes `redis-values.yaml` to override default Helm chart values.  
- This file configures the exporter according to our environment and Redis setup.  

### NOTES!
- **ServiceMonitor** is the link between the exporter and Prometheus.  
- Prometheus uses it to **scrape metrics** from the Redis Exporter endpoint.  
- `redisAddress` specifies the Redis instance that the exporter will connect to.  
- In our setup, **no password is required** because the Redis service from our app is not protected.  

## Install Redis Exporter using Helm

### 1. Add Helm Repositories
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install redis-exporter prometheus-community/prometheus-redis-exporter -f redis-values.yaml
```

### 2. Verify installation:
List installed Helm charts:
```bash
helm ls
```

- Check running pods:
```bash
kubectl get pods
```

- Check ServiceMonitor to confirm Prometheus is scraping the exporter:
```bash
kubectl get servicemonitor
```

Go to Prometheus UI and click on Status - Targets, you can check the Prometheus Redis Exporter.

## Create Alert Rules for Redis

### 1. Observed Alert Conditions
- **Redis is down** → Alert when Redis service is unreachable or not running.  
- **Too many connections** → Alert if Redis connections exceed **90% of capacity**.  

### 2. Prometheus Rule Configuration
- Alert rules are stored in a separate YAML file to allow independent updates:  
`redis-rules.yaml`  

### 3. Reference for Alert Rules
- Use **Awesome Prometheus Alerts** to create Redis alert rules:  
[Prometheus Alerts for Redis](https://samber.github.io/awesome-prometheus-alerts/#redis)  
- Provides predefined alert expressions for Redis metrics.  

### 4. Apply Redis Rules
```bash
kubectl apply -f redis-rules.yaml
```

### 5. Check that the Redis alert rules have been created in the cluster:
```bash
kubectl get prometheusrule
```

## Test Redis Alerts

### 1. Verify Pods
Check the running Redis pods:
```bash
kubectl get pods
```

### 2. Scale Down Deployment:
Edit the Redis deployment and set replicas to 0:
```bash
kubectl edit deployment redis-cart
```

### 3. Confirm Pods Are Terminated
```bash
kubectl get pods
```

### 4. Check Alert in Prometheus

Refresh the Prometheus UI.
The RedisDown alert should be triggered since no Redis pods are running.

## Create Redis Dashboard in Grafana

### 1. Use Existing Dashboard
- Dashboard ID: **763**  
- URL: [Redis Dashboard for Prometheus Redis Exporter](https://grafana.com/grafana/dashboards/763-redis-dashboard-for-prometheus-redis-exporter-1-x/)  

### 2. Import Dashboard in Grafana
1. Open Grafana: `http://<ip-address>:8080`  
2. Go to **Dashboards** → **New Dashboard** → **Import Dashboard**  
3. Paste the **Dashboard ID: 763**  
4. Set **Name**: `Redis Exporter Dashboard`  
5. Select **Prometheus** as the data source  
6. Click **Import**  

### 3. Verify Dashboard Metrics
- Check the Redis Exporter service endpoint in the cluster:
```bash
kubectl describe svc redis-exporter
```

> Use this endpoint to ensure Grafana is receiving metrics from Redis Exporter.
