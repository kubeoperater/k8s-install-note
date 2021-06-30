``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-data
spec:
  capacity:
    storage: 50Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  nfs:
    path: /home/kubernetes_data/jenkins_data
    server: 10.255.72.206
```
