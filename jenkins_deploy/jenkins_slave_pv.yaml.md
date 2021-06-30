``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-slave-data
  namespace: jenkins-project
spec:
  capacity:
    storage: 50Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slaveslow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /home/kubernetes_data/jenkins_data_slave
    server: 10.255.72.206
```
