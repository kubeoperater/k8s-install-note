**1.创建rbac**

> prometheus-rbac.yaml

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-system
```

``` bash
kubectl create -f prometheus-rbac.yaml
```

**2.配置prometheus configmap**

> prometheus.config.yml

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kube-system
data:
  prometheus.yml: |
    global:
      scrape_interval:     10s
      evaluation_interval: 10s
    rule_files:
    - /etc/prometheus/rules.yml

    alerting:
      alertmanagers:
      - kubernetes_sd_configs:
        - role: endpoints
        relabel_configs:
        - action: keep
          regex: altermanager
          source_labels:
          - __meta_kubernetes_service_name
        - action: keep
          regex: kube-system
          source_labels:
          - __meta_kubernetes_namespace
        scheme: http

    scrape_configs:
    - job_name: 'kubernetes-apiservers'
      kubernetes_sd_configs:
      - role: endpoints
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics

    - job_name: 'kubernetes-cadvisor'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor

    - job_name: 'kubernetes-service-endpoints'
      kubernetes_sd_configs:
      - role: endpoints
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name

    - job_name: 'kubernetes-services'
      kubernetes_sd_configs:
      - role: service
      metrics_path: /probe
      params:
        module: [http_2xx]
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-ingresses'
      kubernetes_sd_configs:
      - role: ingress
      relabel_configs:
      - source_labels: [__meta_kubernetes_ingress_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
        regex: (.+);(.+);(.+)
        replacement: ${1}://${2}${3}
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_ingress_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_ingress_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name

    - job_name: 'kubernetes-nodes-physical'
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - source_labels: [__meta_kubernetes_role]
        action: replace
        target_label: kubernetes_role
      - source_labels: [__address__]
        regex: '(.*):10250'
        replacement: '${1}:9100'
        target_label: __address__

  rules.yml: |
    groups:
    - name: node-physical-rules
      rules:
      - alert: 节点内存使用率
        expr: (node_filesystem_size{device="rootfs"} - node_filesystem_free{device="rootfs"}) / node_filesystem_size{device="rootfs"} * 100 > 80
        for: 2m
        labels:
          team: node
        annotations:
          summary: "{{$labels.instance}}: 发现节点根文件系统使用率过高"
          description: "{{$labels.instance}}: 根文件系统使用率超过80% (当前值是: {{ $value }})"

      - alert: 节点内存使用率
        expr: (node_memory_MemTotal - (node_memory_MemFree+node_memory_Buffers+node_memory_Cached )) / node_memory_MemTotal * 100 < 80
        for: 2m
        labels:
          team: node
        annotations:
          summary: "{{$labels.instance}}: 节点内存使用率过高"
          description: "{{$labels.instance}}: 内存使用率超过 80% (当前值是: {{ $value }})"

      - alert: 节点cpu使用率
        expr: node_load1 >10
        for: 2m
        labels:
          team: node
        annotations:
          summary: "{{$labels.instance}}: 节点一分钟负载超过10%"
          description: "{{$labels.instance}}: 节点一分钟负载超过10% (当前值是: {{ $value }})"


---
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
      group_wait: 30s
      group_interval: 1m
      repeat_interval: 1m
      routes:
      - receiver: webhook
        group_wait: 10s
    receivers:
    - name: webhook
      webhook_configs:
      - url: 'http://10.255.72.206:80/'
        send_resolved: true
```

``` bash
kubectl create -f prometheus.config.yml
```

**3.prometheus.deploy.yaml**

> prometheus.deploy.yaml

``` yaml
---
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
            path: /prometheus-data/
            type: DirectoryOrCreate
        - name: config-volume
          configMap:
            name: prometheus-config
```

``` bash
kubectl create -f prometheus.deploy.yaml
```

**4.prometheus.service.yaml**

> prometheus.service.yaml

``` yaml
---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: prometheus
  name: prometheus
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 9090
    targetPort: 9090
    nodePort: 30003
  selector:
    app: prometheus
```

``` bash
kubectl create -f prometheus.service.yaml
```

**5.node_exporter_daemonset.yaml**

>node_exporter_daemonset.yaml #node节点监控

``` yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace : kube-system
spec:
  template:
    metadata:
      labels:
        app: node-exporter
      name: node-exporter
    spec:
      hostNetwork: true
      hostPID: true
      containers:
      - image: hub-dev.example.com/prometheus/node-exporter:v0.15.2
        args:
        - "--path.procfs=/host/proc"
        - "--path.sysfs=/host/sys"
        name: node-exporter
        ports:
        - containerPort: 9100
          hostPort: 9100
          name: scrape
        resources:
          requests:
            memory: 30Mi
            cpu: 100m
          limits:
            memory: 50Mi
            cpu: 200m
        volumeMounts:
        - name: proc
          readOnly:  true
          mountPath: /host/proc
        - name: sys
          readOnly: true
          mountPath: /host/sys
      tolerations:
        - effect: NoSchedule
          operator: Exists
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

``` bash
kubectl create -f node_exporter_daemonset.yaml
```

**6.ingress-prometheus.yaml**

> ingress-prometheus.yaml

``` yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus-dashboard
  namespace: kube-system
spec:
  rules:
  - host: prometheus-dashboard.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: prometheus
          servicePort: 9090
```
