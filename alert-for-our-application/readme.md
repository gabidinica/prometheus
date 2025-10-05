# Demo Project: Configure Alerting for Our Application

## Technologies Used
- Prometheus  
- Kubernetes  
- Linux  

## Project Description
- Configure the **Monitoring Stack** to notify whenever:  
  - **CPU usage > 50%**  
  - **Pod cannot start**  
- Define **Alert Rules** in the Prometheus server  
- Configure **Alertmanager** with an **Email Receiver** for notifications  

---

## Understanding Alerts in Prometheus UI

- In the **Prometheus UI**, click on **Alerts** to view configured alert rules.  
- **Green color** → Alert is **inactive** (condition not met).  
- **Red color** → Alert is **firing** (condition met).  
  - **Firing** means the alert is sent to **Alertmanager**.  

### Alert Expression
An alert expression in Prometheus consists of:
- A **Prometheus function**  
- A **metric**  
- A **filtered label** with key-value pairs  

## Alert: CPU Usage > 50%

### Metric Used
- `node_cpu_seconds_total` → represents CPU time.  
- The `mode="idle"` value indicates when the CPU is **not** being used.  

### Logic
- Calculate the **average rate** over the last **2 minutes**.  
- Subtract **idle time** from total CPU to determine actual usage per instance.  
- If CPU usage remains above **50%** for **2 minutes**, trigger the alert.  
- Set severity to **warning**.  
- Value = {{ $value }} gives the actual value of the usage
- Instance = {{ $labels.instance }} instance that’s affected
- Runbook_url is removed, but if it describes the issue and possible fix for it, then it can be added.

## Prometheus Operator and Custom Alerts

The **Prometheus Operator** allows us to create custom Kubernetes components that define alert rules.  
These custom resources are then picked up by Prometheus, which applies the new alerting configuration.  

### PrometheusRule Resource
- Used to define custom alerting rules in Kubernetes.  
- Official reference: [PrometheusRule Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.15/html/monitoring_apis/prometheusrule-monitoring-coreos-com-v1)  

## Applying Alert Rules

### 1. Trigger Alert Immediately
- Configure the alert with `for: 0m` so it fires as soon as the condition is met.  

### 2. Apply Alert Rules to Cluster
In the terminal, run:
```bash
kubectl apply -f alert-rules.yaml
```

### 3. Verify Prometheus Rule
```bash
kubectl get PrometheusRule -n monitoring
```

### 4. Open the **Prometheus UI**.  
- Navigate to **Rules** to confirm that the new alert rule has been added.  

### 5. Verify Pods
Check that the Prometheus pods are running in the `monitoring` namespace:
```bash
kubectl get pods -n monitoring
```

### 6. Confirm Configuration Reload
Check the logs of the Prometheus config reloader container to ensure the new rules were applied:
```bash
kubectl logs prometheus-kube-prometheus-prometheus-0 -n monitoring -c config-reloader
```

## Test Alert Rule for CPU Load

### 1. Verify Query in Grafana
- Open the **Grafana UI**.  
- Edit the **CPU Utilisation** panel.  
- Paste the alert query (`CPU > 50%`) into the **Metric browser** and click **Run Query**.  
- Confirm that the panel shows expected results.  

### 2. Generate CPU Load with cpustress
We will use the `cpustress` image from Docker Hub:  
[containerstack/cpustress](https://hub.docker.com/r/containerstack/cpustress)

### 3. Run CPU Stress Pod
Translate the Docker command to a Kubernetes pod using `kubectl run`:
```bash
kubectl run cpu-test --image=containerstack/cpustress -- --cpu 4 --timeout 30s --metrics-brief
```

> This should trigger the CPU > 50% alert and send it to Alertmanager.

Make sure the stress pod is running:
```bash
kubectl get pod
```

### 4. Check Alert State in Prometheus
- In the Prometheus UI, go to Alerts.
- The alert should first appear as Pending for 2 minutes (as per the alert rule).
- After 2 minutes, it will change to Firing.

## Access Alertmanager

Forward the Alertmanager service port to your local machine:
```bash
kubectl port-forward svc/monitoring-kube-prometheus-alertmanager -n monitoring 9093:9093 &
```

- Open http://ip-address:9093 in your browser to view active alerts.

### 1. View Configuration
- In the **Alertmanager UI**, click on **Status** → **Configuration**.  
- This displays the current Alertmanager setup, including receivers and routing rules.  

### 2. Receivers
- **Receivers** define where Alertmanager sends alerts received from Prometheus.  
- Examples: Email, Slack, PagerDuty, Webhooks, etc.  

### 3. Inhibit Rules
- **Inhibit rules** suppress notifications for certain alerts when other related alerts are already firing.  
- Useful to avoid alert noise and redundant notifications.  

### 4. Retrieve Alertmanager Secret
The Alertmanager configuration is stored as a Kubernetes secret:
```bash
kubectl get secret alertmanager-monitoring-kube-prometheus-alertmanager-generated -n monitoring -o yaml | less
```

### 5. Decode and Unarchive Secret
Copy the encoded_content from the secret and decode it:
```bash
echo "encoded_content" | base64 -d | gunzip | less
```

## Configure Email Notifications in Alertmanager

### 1. Reference Documentation
We will use the **Monitoring API** to configure Alertmanager email notifications:  
[AlertmanagerConfig API](https://docs.redhat.com/en/documentation/openshift_container_platform/4.12/html/monitoring_apis/alertmanagerconfig-monitoring-coreos-com-v1beta1)  

### 2. Alertmanager Configuration File
- Email receiver is configured in `alert-manager-configuration.yaml`.  
- Replace the email address with your own in the YAML file.  

### 3. repeatInterval
- Specifies how often the alert will be sent if the issue persists.
- In this case, 30 minutes between repeated alert emails until the issue is resolved.
## Store Email Password in a Kubernetes Secret

### Create Local Secret File
Create a file named `email-secret.yaml` **locally** (do not commit to the repository) with the following content:

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: gmail-auth
  namespace: monitoring
data:
  password: <base-64-encoded-value-of-your-password>
  authIdentity: <your_email_address_here>
```

> password → Base64 encoded value of your email password.
> authIdentity → Your email address.

### 1. Create an App Password
- For Gmail accounts with **2-Step Verification**, generate an application-specific password:  
[Google App Passwords](https://myaccount.google.com/apppasswords)  
- Name the password: `prometheus-alerts`  
- Copy the displayed password; it will be used in the `email-secret.yaml` file.  

### 2. Apply Email Secret and Alertmanager Configuration
```bash
kubectl apply -f email-secret.yaml
kubectl apply -f alert-manager-configuration.yaml
```

### 3. Verify Alertmanager Configuration:
```bash
kubectl get alertmanagerconfig -n monitoring
kubectl get pods -n monitoring
```

> Check the Alertmanager UI and refresh the page to see updated configuration.

### 4. Test Email Notification

- Delete the existing CPU stress pod:
```bash
kubectl delete pod cpu-test
```

- Run a new CPU stress pod to trigger the alert:
```bash
kubectl run cpu-test --image=containerstack/cpustress -- --cpu 4 --timeout 60s --metrics-brief
```

> Verify the alert in Prometheus UI (should be firing).

- Check active alerts in Alertmanager via JSON endpoint:
``` bash
http://<IP-address>:9093/api/v2/alerts
```

> Confirm receipt of the alert in your email inbox.
