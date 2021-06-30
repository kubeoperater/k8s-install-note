``` yaml
kind: Service
apiVersion: v1
metadata:
  name: jenkins-slave-slb
  namespace: jenkins-project
spec:
  selector:
    k8s-app: jenkins
  ports:
    - protocol: TCP
      port: 50000
      targetPort: 50000
  type: NodePort
  externalTrafficPolicy: Local
```
