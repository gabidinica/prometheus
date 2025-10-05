# Demo Project: Configure Monitoring for Own Application

## Technologies Used
- Prometheus  
- Kubernetes  
- Node.js  
- Grafana  
- Docker  
- Docker Hub  

## Project Description
- Configure our **Node.js application** to collect and expose metrics using the **Prometheus Client Library**  
- Deploy the Node.js application with a **metrics endpoint** into the Kubernetes cluster  
- Configure **Prometheus** to scrape the exposed metrics  
- Visualize application metrics in a **Grafana Dashboard**  

---

## Monitor Node.js Application

### 1. Clone Node.js App
The application can be cloned from GitLab:  
[Node.js App for Monitoring](https://gitlab.com/twn-devops-bootcamp/latest/16-prometheus/nodejs-app-monitoring)  

### 2. Use Prometheus Client Library
- Prometheus Client Libraries are language-specific libraries to expose metrics.  
- For Node.js, the **prom-client** library is used.  

### 3. Expose Metrics
- The Node.js app exposes a **metrics endpoint** for Prometheus to scrape.  
- Start the application:
```bash
node app/server.js
```

> Open browser and paste: `localhost:3000`
> The metrics endpoint is usually accessible at /metrics and can be verified in the browser.

## Metrics Tracked in Node.js Application

### 1. Metrics
In `server.js`, the Node.js app tracks the following metrics:

1. **Number of requests** → Counts the total HTTP requests received.  
2. **Duration of requests** → Measures the latency or time taken to process requests.  

### 2. Prometheus Client Library
- The library used for Node.js is **prom-client**  
- Documentation and usage examples: [prom-client on NPM](https://www.npmjs.com/package/prom-client)  

### 3. How It Works
- The app exposes metrics at the `/metrics` endpoint.  
- Prometheus scrapes this endpoint to collect data and visualize it in Grafana.  

> client.collectDefaultMetrics: default metrics recommended by Prometheus are collected

## Build and Push Node.js Docker Image

### 1. Build Docker Image
In terminal:
```bash
docker build -t <private_repo_name>:nodeapp .
```

### 2. Login to Docker Hub
```bash
docker login
```

### Push Docker Image
```bash
docker push <private_repo_name>:nodeapp
```

> Replace <private_repo_name>:nodeapp with the name of your image

### 4. Verify in Docker Hub
- Go to your Docker Hub repository
- Click on the Tags tab
- Check that the Node.js app image is available

## Deploy Node.js App into Kubernetes Cluster

### 1. Update Deployment Configuration
- Edit `config.yaml` to replace the image with your Docker Hub image:
```yaml
image: <private_repo_name>:nodeapp
```

### 2. Create Docker Registry Secret
- This secret allows Kubernetes to pull your private Docker image:
```bash
kubectl create secret docker-registry my-registry-key \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<your_username> \
  --docker-password=<your-docker-password>
```

Replace <your_username> and <your-docker-password> with your Docker Hub credentials.

### 3. Apply Deployment and Service
```bash
kubectl apply -f config.yaml
```

> The Node.js app will be deployed in the cluster, using the private Docker image.

### 4. Check pods and services
```bash
kubectl get pods
kubectl get svc
```

### 5. Port-Forward to Access App
```bash
kubectl port-forward svc/nodeapp 3000:3000
```

## Create ServiceMonitor for Node.js App

### 1. ServiceMonitor Configuration
- The `ServiceMonitor` resource is defined in `k8s-config.yaml`.  
- **Port attribute** → Set to the name of the service port of your Node.js app.  
- **Selector** → Configured to find the app with label `nodeapp`.  
- **Namespace** → Default namespace.  
- **Metrics Endpoint** → Access the `/metrics` endpoint on the service port. 

### 2. Apply Configuration
`kubectl apply -f k8s-config.yaml`

### 3. Open Prometheus UI
- URL: `http://<IP>:9090`  

### 4. Check Targets
- Navigate to **Status → Targets**  
- Look for the target corresponding to your Node.js app:  
  - **Name:** `monitoring-node-app`  
- Ensure the target is **UP** and metrics are being scraped successfully. 

## Create Grafana Dashboard for Node.js App

### 1. Open Grafana
- URL: `http://<IP>:8080`  
- Go to **Dashboards → New Dashboard → Add Visualization**  

### 2. Query Prometheus for Metrics
- In a new tab, open **Prometheus UI**  
- Execute the query to calculate the number of total operations in the last 2 minutes:
```promql
rate(http_request_operations_total[2m])
```

> Use this query in Grafana to create a visualization for request rate.

### 1. Add Query to Dashboard
- Copy the Prometheus query:
```promql
rate(http_request_operations_total[2m])
```

### 2. Save Dashboard
> Save the dashboard with the name: Nodeapp Telemetry

### 3. Edit Panel
- Edit the visualization panel
- Change the panel title to: Requests Per Second

### 4. Add New Visualization
- In the **Nodeapp Telemetry** dashboard, click **Add → Visualization**  

### 5. Add Query
- Copy the Prometheus metric:
```promql
rate(http_request_duration_seconds_sum[2m])
```

- Paste it into the Grafana query field and click Run Query

### 6. Set panel title to: Request Duration

### 7. Save the updated Nodeapp Telemetry dashboard with the new panel included. 
