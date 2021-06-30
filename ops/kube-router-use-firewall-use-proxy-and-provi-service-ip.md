[参考连接1](http://blog.51cto.com/net592/2059315)

[https://github.com/cloudnativelabs/kube-router/tree/master/daemonset](https://github.com/cloudnativelabs/kube-router/tree/master/daemonset

**1.本篇文章是在解决两个问题**
- 1. 实现内网dns转发
- 2. k8s内部的service ip 可以被内网访问

**2.使用kube-router 代理kube-proxy,不再使用calico,而使用kube-router做网络bgp**

# 需要注意两点：

- 1. /etc/cni/net.d/ 这个目录 如果不懂，保持目录为空
- 2. /var/lib/kube-router/kubeconfig 是一个文件，内容为之前创建的kube-proxy.kubeconfig
- 3. controller-manager 需要添加一个参数：
--allocate-node-cidrs=true

- 1. rbac配置：

> kube-router-rbac.yaml

``` yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: kube-router
  namespace: kube-system
rules:
  - apiGroups:
    - ""
    resources:
      - namespaces
      - pods
      - services
      - nodes
      - endpoints
    verbs:
      - list
      - get
      - watch
  - apiGroups:
    - "networking.k8s.io"
    resources:
      - networkpolicies
    verbs:
      - list
      - get
      - watch
  - apiGroups:
    - extensions
    resources:
      - networkpolicies
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: kube-router
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-router
subjects:
- kind: User
  name: system:kube-proxy
  namespace: kube-system
```

- 2. 配置deployment和创建service

>kube-router-all-service-daemonset-advertise-routes.yaml

``` yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-router-cfg
  namespace: kube-system
  labels:
    tier: node
    k8s-app: kube-router
data:
  cni-conf.json: |
    {
      "name":"kubernetes",
      "type":"bridge",
      "bridge":"kube-bridge",
      "isDefaultGateway":true,
      "ipam": {
        "type":"host-local"
      }
    }
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: kube-router
  namespace: kube-system
  labels:
    k8s-app: kube-router
spec:
  template:
    metadata:
      labels:
        k8s-app: kube-router
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      containers:
      - name: kube-router
        image: cloudnativelabs/kube-router
        args:
          - "--run-router=true"
          - "--run-firewall=true"
          - "--run-service-proxy=true"
          - "--kubeconfig=/var/lib/kube-router/kubeconfig"
          - "--advertise-cluster-ip=true"
          - "--nodes-full-mesh=false" #各个node之间不互相建立bgp
          - "--cluster-asn=65000" #这是自己的asn号
          - "--peer-router-ips=192.168.0.254" #目标交换机的地址
          - "--peer-router-asns=65000" #目标交换机的asn号
        securityContext:
          privileged: true
        imagePullPolicy: Always
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        livenessProbe:
          httpGet:
            path: /healthz
            port: 20244
          initialDelaySeconds: 10
          periodSeconds: 3
        volumeMounts:
        - name: lib-modules
          mountPath: /lib/modules
          readOnly: true
        - name: cni-conf-dir
          mountPath: /etc/cni/net.d
        - name: kubeconfig
          mountPath: /var/lib/kube-router/kubeconfig
          readOnly: true
      initContainers:
      - name: install-cni
        image: busybox
        imagePullPolicy: Always
        command:
        - /bin/sh
        - -c
        - set -e -x;
          if [ ! -f /etc/cni/net.d/10-kuberouter.conf ]; then
            TMP=/etc/cni/net.d/.tmp-kuberouter-cfg;
            cp /etc/kube-router/cni-conf.json ${TMP};
            mv ${TMP} /etc/cni/net.d/10-kuberouter.conf;
          fi
        volumeMounts:
        - name: cni-conf-dir
          mountPath: /etc/cni/net.d
        - name: kube-router-cfg
          mountPath: /etc/kube-router
      hostNetwork: true
      tolerations:
      - key: CriticalAddonsOnly
        operator: Exists
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
        operator: Exists
      volumes:
      - name: lib-modules
        hostPath:
          path: /lib/modules
      - name: cni-conf-dir
        hostPath:
          path: /etc/cni/net.d
      - name: kube-router-cfg
        configMap:
          name: kube-router-cfg
      - name: kubeconfig
        hostPath:
          path: /var/lib/kube-router/kubeconfig
```

kube-router 查看bgp 邻居

```
gobgp  neighbor
```

**3.做内网和k8s内部coredns之间的转发**

#这一项也是在kube-router发布了service的ip之后，就可以将coredns的service ip 和局域网内部的dnsserver 做相互转发。


- 安装bind模拟内网dnsserver：

``` bash
yum install bind bind-utils -y
```

- 注释和确认有以下配置


> /etc/named.config

```
dnssec-enable no;
dnssec-validation no;
```
> /etc/named.rfc1912.zones

``` yaml

zone "cluster.local." {
        type forward;
        forwarders{
        coredns-service-ip;
        };
};
```


coredns 上的配置：

>Corefile配置：

``` yaml
proxy example.inc 10.255.0.1:53 10.255.0.22:53
```
