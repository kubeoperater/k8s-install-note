**1.部署prometheus-webhook-dingtalk**

* 这里使用github的一个项目prometheus-webhook-dingtalk部署钉钉机器人的消息转发

``` yaml
---
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  labels:
    name: prometheus-webhook-dingtalk
  name: prometheus-webhook-dingtalk
  namespace: kube-system
spec:
  replicas: 2
  selector:
    matchLabels:
      app: prometheus-webhook-dingtalk
  template:
    metadata:
      labels:
        app: prometheus-webhook-dingtalk
    spec:
      containers:
      - image: hub-dev.example.com/prometheus/prometheus-webhook-dingtalk:v0.3.0
        name: prometheus-webhook-dingtalk
        args:
        - "--ding.profile=node=https://oapi.dingtalk.com/robot/send?access_token=xxxxx"
        - "--web.listen-address=:8080"
        ports:
        - containerPort: 8080
          protocol: TCP
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 500m
            memory: 2500Mi
      imagePullSecrets:
        - name: IfNotPresent
```

**2.配置prometheus-webhook-dingtalk成为k8s服务**

* 这里偷懒不写yaml文件直接用命令由deployment生成service项

```bash
kubectl expose deployment prometheus-webhook-dingtalk --port=80 --target-port=8080 --name=prometheus-webhook-dingtalk -n kube-system
```

**3.配置altermanager的webhook的配置**

``` yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: alertmanager
  namespace: kube-system
data:
  config.yml: |-
    global:
      resolve_timeout: 5m
    route:
      receiver: webhook
      group_by: ['alertname', 'cluster']

      group_wait: 30s
      group_interval: 5m
      repeat_interval: 3h
      routes:
      - match:
          team: node
        receiver: webhook_node
        group_wait: 10s
    receivers:
    - name: webhook_node
      webhook_configs:
      - url: 'http://prometheus-webhook-dingtalk/dingtalk/node/send'
        send_resolved: true
    - name: webhook
      webhook_configs:
      - url: 'http://prometheus-webhook-dingtalk/dingtalk/test/send'
        send_resolved: true
```
