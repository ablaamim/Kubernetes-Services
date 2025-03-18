### RETRIEVE GRAFANA PASSWORD :

```
[root@master Prometheus-gpu]# kubectl get secret rancher-monitoring-grafana -n cattle-monitoring-system -o jsonpath="{.data.admin-password}" | base64 --decode; echo

```

prom-operator
