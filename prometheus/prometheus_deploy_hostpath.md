**1.设置prometheus的deploy hostpath**

* hostpath的解释参照官方：[hostPath](https://kubernetes.io/docs/concepts/storage/volumes/)

** 设置hostpath

``` yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  labels:
    name: prometheus-deployment
  name: prometheus
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      securityContext:
        runAsUser: 1000
        fsGroup: 2000
        runAsNonRoot: true
      serviceAccountName: prometheus
      containers:
      - image: hub-dev.example.com/prometheus/prometheus:v2.0.0
        name: prometheus
        command:
        - "/bin/prometheus"
        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        - "--storage.tsdb.path=/prometheus"
        - "--storage.tsdb.retention=24h"
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: "/prometheus"
          name: prometheus-data
        - mountPath: "/etc/prometheus"
          name: config-volume
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 500m
            memory: 2500Mi
      imagePullSecrets:
        - name: regsecret
      volumes:
      - name: prometheus-data
        hostPath:
          path: /prometheus-data/ #提前创建这个目录 并赋予777 权限 [这里参考官网的pv和pvc的创建，使用nfs或者其他网络存储来实现数据落地](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
          type: DirectoryOrCreate #如果目录不存在则创建

      - name: config-volume
        configMap:
          name: prometheus-config
```
