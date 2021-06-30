# k8s部署完成之后的一些梳理和维护工作

整体架构图:

![架构1](./kubernetes-high-level-component-archtecture.jpg)

---

![架构2](./kubernetes-架构.png)

## Kubernetes主要由以下几个核心组件组成：

 * etcd保存了整个集群的状态；
 * apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
 * controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
 * scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
 * kubelet负责维护容器的生命周期，同时也负责Volume（CSI）和网络（CNI）的管理；
 * kube-proxy负责为Service提供cluster内部的服务发现和负载均衡；
 * kube-router 是kube-proxy的替代 更稳定,使用lvs做服务的发现和负载均衡以及网络的管理
 * Calico 是一个纯三层的虚拟网络方案，Calico 为每个容器分配一个 IP，每个 host 都是 router，把不同 host 的容器连接起来。与 VxLAN 不同的是，Calico 不对数据包做额外封装，不需要 NAT 和端口映射，扩展性和性能都很好。与其他容器网络方案相比，Calico 还有一大优势：network policy。用户可以动态定义 ACL 规则，控制进出容器的数据包，实现业务需求。

## 除了核心组件，还有一些推荐的Add-ons：

  * coredns 负责为整个集群提供DNS服务
  * Ingress nginx 为服务提供外网入口和代理
  * prometheus 提供资源监控
  * Dashboard 提供GUI
