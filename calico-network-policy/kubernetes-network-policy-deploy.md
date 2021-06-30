强调一点：
如果pod的ip需要固定并且需要使用svc，则 需要使用cni.projectcalico.org/ipAddrsNoIpam， 不加NoIpam，svc不通。
同时 该策略生效范围是集群内部，pod的策略可以被集群外部访问，svc的策略开放只是对集群内部，拒绝是所有都拒绝。

1. 参考文档
  - https://www.kubernetes.org.cn/network-policy
  - https://segmentfault.com/a/1190000012692009
  - https://www.kubernetes.org.cn/1909.html
  - https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/calico
  - https://docs.projectcalico.org/v3.0/reference/calicoctl/resources/networkpolicy


2. 在10.5版本里,要想在Kubernetes集群中使用Calico进行网络隔离，必须满足以下条件：
    - kube-apiserver必须开启运行时extensions/v1beta1/networkpolicies，即设置启动参数：–runtime-config=extensions/v1beta1/networkpolicies=true #这一步在10.5的版本里 验证不需要配置也可以。
    - kubelet必须启用cni网络插件，即设置启动参数：–network-plugin=cni
    - kube-proxy（kube-router)必须启用iptables代理模式，这是默认模式，可以不用设置
    - kube-proxy(kube-router)不得启用–masquerade-all，这会跟calico冲突

3. 创建一个空间做好默认的策略限制：

>ns-production.yaml

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  annotations:
    net.beta.kubernetes.io/network-policy: |
      {
        "ingress": {
          "isolation": "DefaultDeny"
        }
      }
```

4. 部署一个nginx应用：

> nginx-deploy-service.yaml  

``` yaml
kind: Deployment
apiVersion: apps/v1beta2
metadata:
  labels:
    k8s-app: nginx
  name: nginx
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: nginx
  template:
    metadata:
      labels:
        k8s-app: nginx
    spec:
      containers:
      - name: nginx
        image: hub-dev.example.com/base/nginx:latest
        ports:
        - containerPort: 80
          protocol: TCP
        livenessProbe:
          httpGet:
            scheme: HTTP
            path: /
            port: 80
          initialDelaySeconds: 30
          timeoutSeconds: 30
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: nginx-lb
  name: nginx-lb
  namespace: production
spec:
  ports:
    - port: 80
      targetPort: 80
  selector:
    k8s-app: nginx
```

5. 部署一个jetty应用：

> jetty-deploy-service.yaml

``` yaml
kind: Deployment
apiVersion: apps/v1beta2
metadata:
  labels:
    k8s-app: jetty
  name: jetty
  namespace: production
spec:
  replicas: 2
  selector:
    matchLabels:
      k8s-app: jetty
  template:
    metadata:
      labels:
        k8s-app: jetty
    spec:
      containers:
      - name: jetty
        image: hub-dev.example.com/base/jetty:latest
        ports:
        - containerPort: 8080
          protocol: TCP
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: jetty-lb
  name: jetty-lb
  namespace: production
spec:
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    k8s-app: jetty
```

6. 进入到nginx容器里访问jetty的lb,此时是不通的。

``` bash
telnet jetty-lb 8080
```

7. 配置nginx访问jetty的策略，再访问舅可以通了

>policy-allow-nginx-access-jetty.yaml

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx-access-jetty
  namespace: production
spec:
  podSelector:
    matchLabels:
      k8s-app: jetty
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: production
    - podSelector:
        matchLabels:
          k8s-app: nginx
    ports:
    - protocol: TCP
      port: 8080
```
