``` yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins-project
  labels:
    k8s-app: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: jenkins
  template:
    metadata:
      labels:
        k8s-app: jenkins
    spec:
      containers:
      - name: jenkins
        image: hub-dev.example.com/jenkins/jenkins:2.107.3
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
          readOnly: false
        ports:
        - containerPort: 8080
        - containerPort: 50000
      volumes:
        - name: jenkins-home
          persistentVolumeClaim:
            claimName: jenkins-data-pvc
      serviceAccount: "jenkins-admin"
```
