**1.配置服务监控规则**

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor #prometheus operation 配置的类名
metadata:
  name: example-app
  labels:
    team: frontend
spec:
  selector:
    matchLabels:
      app: example-app
  endpoints:
  - port: web
```

**2.配置报警规则**

``` yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule #prometheus operation 配置的类名
metadata:
  creationTimestamp: null
  labels:
    prometheus: example
    role: alert-rules
  name: prometheus-example-rules
spec:
  groups:
  - name: ./example.rules
    rules:
    - alert: ExampleAlert
      expr: vector(1)
```
