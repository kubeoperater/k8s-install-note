``` yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: jenkins
  name: jenkins-admin
  namespace: jenkins-project

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
  labels:
    k8s-app: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: jenkins-admin
  namespace: jenkins-project

---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  labels:
    k8s-app: jenkins
  name: jenkins-admin
rules:
- apiGroups: ["", "extensions", "apps"]
  resources:
    - nodes
    - nodes/proxy
    - endpoints
    - secrets
    - pods
    - deployments
    - services
  verbs: ["get", "list", "watch"]
```
