``` yaml
kind: Service
apiVersion: v1
metadata:
  name: jenkins-slb
  namespace: jenkins-project
spec:
  selector:
    k8s-app: jenkins
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
