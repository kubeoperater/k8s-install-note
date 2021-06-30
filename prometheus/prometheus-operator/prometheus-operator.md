**1.prometheus-operator git 地址：**

[https://github.com/coreos/prometheus-operator](https://github.com/coreos/prometheus-operator)

**2.部署prometheus-operator**

``` bash
git clone https://github.com/coreos/prometheus-operator.git
cd prometheus-operator
#创建namespace monitoring
kubectl create -f contrib/kube-prometheus/manifests/00namespace-namespace.yaml
修改namespace为 monitoring
修改bundle.yaml里面的image地址
kubectl create -f bundle.yaml
```

**3.部署prometheus**

``` bash
cd prometheus-operator/contrib/kube-prometheus/manifests/
grep "image: quay.io" ./*.yaml
将输出的镜像地址全部改成自己相应的私仓的地址和对应的版本号 这里需要用到科学上网。
kubectl apply -f *.yaml
完成prometheus的部署
```

**4.这里使用的是ingress-nginx做的前端负载所以这里还做了prometheus和grafana的代理**

``` yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus-dashboard
  namespace: monitoring
spec:
  rules:
  - host: prometheus-dashboard.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: prometheus-k8s
          servicePort: 9090
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana-dashboard
  namespace: monitoring
spec:
  rules:
  - host: grafana-dashboard.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: grafana
          servicePort: 3000
```
