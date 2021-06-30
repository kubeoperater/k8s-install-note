``` yaml
livenessProbe:
  httpGet:
    path: /starott_cloud_client/test/overview-frontend
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 5
  timeoutSeconds: 1
```
