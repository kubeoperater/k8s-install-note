### 遗留的问题，  accessModes: ReadWriteOnce 造成 replica只能为1，缺乏高可用和集群功能。

**1.配置alertmanager的pv**

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: alertmanager-pv
  labels:
    app: alertmanager-pv
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: alertmanager-slow
  nfs:
    server: 10.255.72.206
    path: "/home/kubernetes_data/alertmanager_data"
```

**2.配置alertmanager的部署配置文件**

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: Alertmanager
metadata:
  labels:
    alertmanager: main
  name: main
  namespace: monitoring
spec:
  baseImage: hub-dev.example.com/prometheus/alertmanager
  nodeSelector:
    beta.kubernetes.io/os: linux
  replicas: 1
  storage:
    volumeClaimTemplate:
      spec:
        volumeMode: Filesystem
        storageClassName: alertmanager-slow
        selector:
          matchLabels:
            app: alertmanager-pv
        resources:
          requests:
            storage: 10Gi
  serviceAccountName: alertmanager-main
  version: v0.14.0
```
