**1.创建pv**

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: prometheus-pv
  labels:
    app: prometheus-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  nfs:
    server: 10.255.72.206
    path: "/home/kubernetes_data/prometheus_data"
```

**2.修改prometheus配置**

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  labels:
    prometheus: k8s
  name: k8s
  namespace: monitoring
spec:
  alerting:
    alertmanagers:
    - name: alertmanager-main
      namespace: monitoring
      port: web
  baseImage: hub-dev.example.com/prometheus/prometheus_example
  nodeSelector:
    beta.kubernetes.io/os: linux
  replicas: 1
  resources:
    requests:
      memory: 400Mi
  ruleSelector:
    matchLabels:
      prometheus: k8s
      role: alert-rules
  storage:
    volumeClaimTemplate:
      spec:
        volumeMode: Filesystem
        storageClassName: slow
        selector:
          matchLabels:
            app: prometheus-pv
        resources:
          requests:
            storage: 10Gi
  serviceAccountName: prometheus-k8s
  #serviceMonitorSelector:
  #  matchExpressions:
  #    - key: k8s-app
  #      operator: Exists
  version: v2.2.1
```
