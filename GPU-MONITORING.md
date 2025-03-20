## RETRIEVE GRAFANA PASSWORD :

### Display grafana password :

```sh
# kubectl get secret rancher-monitoring-grafana -n cattle-monitoring-system -o jsonpath="{.data.admin-password}" | base64 --decode; echo

# prom-operator
```

### Overview :

> The NVIDIA Data Center GPU Manager (DCGM) exporter gathers metrics from your GPUs (such as utilization, memory usage, temperature, and fan speed) and exposes them on an HTTP endpoint. With a proper ServiceMonitor, Prometheus will scrape these metrics and make them available for Grafana dashboards. This enables you to continuously monitor your GPU resources in a Kubernetes environment managed by Rancher.



### Configuring Prometheus to Scrape DCGM Metrics :

* Create a Service for the DCGM Exporter

If the GPU Operator did not automatically create one, define a Service that targets the DCGM exporter pods:

```
apiVersion: v1
kind: Service
metadata:
  name: nvidia-dcgm-exporter
  namespace: gpu-operator
  labels:
    app: nvidia-dcgm-exporter
spec:
  ports:
    - name: metrics
      port: 9400
      targetPort: 9400
  selector:
    app: nvidia-dcgm-exporter
```

* Apply this manifest using:

```sh
kubectl apply -f dcgm-exporter-service.yaml
```

* Create a ServiceMonitor for Prometheus :

Define a ServiceMonitor so that Prometheus knows where and how to scrape the DCGM exporter:

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nvidia-dcgm-exporter
  namespace: cattle-monitoring-system  # Prometheus Operator’s namespace
spec:
  selector:
    matchLabels:
      app: nvidia-dcgm-exporter
  namespaceSelector:
    matchNames:
      - gpu-operator  # Where the DCGM exporter service is deployed
  endpoints:
    - port: metrics
      interval: 10s
      path: /metrics
```

* Apply the ServiceMonitor :

```bash
kubectl apply -f dcgm-exporter-servicemonitor.yaml
```

*  Integrating DCGM Metrics into Grafana

*  Create/Import GPU Dashboards :

Import a Pre-made Dashboard:
Visit Grafana Dashboards and search for GPU dashboards (e.g., dashboards designed for NVIDIA GPUs). Download the JSON model and then import it into Grafana.

Build a Custom Dashboard:
Log in to Grafana (default credentials for Rancher Monitoring are typically admin/prom-operator) and create a new dashboard. Add panels that query GPU metrics 

such as:

```
dcgm_gpu_utilization
dcgm_memory_used
dcgm_gpu_temp
dcgm_gpu_fan_speed
```