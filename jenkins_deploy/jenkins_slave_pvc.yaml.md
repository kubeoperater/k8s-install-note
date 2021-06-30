``` yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-slave-data-pvc
  namespace: jenkins-project
spec:
  accessModes:
    - ReadWriteMany 
  volumeMode: Filesystem
  resources:
    requests:
      storage: 50Gi
  storageClassName: slaveslow
```
