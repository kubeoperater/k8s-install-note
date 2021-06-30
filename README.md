**k8s 安装文档**

**一些简介**

* 本文档采用k8s v1.10.0二进制的集群部署方式，主要更改 使用kube-router 代理kube-proxy,使用ingress-nginx做边缘负载,使用haproxy+heartbeat实现高可用

	* 本文档持续更新，后续将继续深入了解prometheus，helm等组件，已经投产之后的一些故障和高可用方案。

  #一些可参考的网站和文档
  * [rbac](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
  * [创建用户](https://github.com/rootsongjc/kubernetes-handbook/blob/master/guide/kubectl-user-authentication-authorization.md)
	# 目录
	  * [k8s v.1.10.0 二进制安装](./1.k8s v.1.10.0 二进制安装.md)
	  * [calico网络设置](./2.k8s 二进制安装集群-calico网络设置.md)
	  * [calico网络策略](./calico-network-policy/kubernetes-network-policy-deploy.md)
	  * [kubedns配置](./3.K8S 二进制安装集群-kubedns配置.md)
	  * [kube-dashboard](./4.K8S 二进制安装集群-kube-dashboard.md)
	  * [nginx-ingress](./5.KUBERNETES V1.10.0 二进制安装 之nginx-ingress 边界路由安装配置.md)
	  * [kube-route](./6.kube-route 代替kube-proxy.md)
	  * [node部署初始化脚本](./7.node部署初始化脚本.md)
	  * [node批量部署的api账户一些自动策略设置](./8.node批量部署的api账户一些自动策略设置.md)
	  * [haproxy + keepalived 配置ingress-nginx 的前端代理](./9.haproxy + keepalived 配置ingress-nginx 的前端代理.md)
	  * [prometheus部署](./prometheus/prometheus-index.md)
	    * [prometheus operation配置服务监控项和报警项](./prometheus/prometheus_configure/prometheus配置报警规则.md)
	    * [alertmanager-use-nfs](./prometheus/prometheus-operator/alertmanager-use-nfs.md)
	    * [prometheus_use_nfs](./prometheus/prometheus-operator/prometheus_use_nfs.md)
	    * [prometheus-custom-configure-rules](./prometheus/prometheus-operator/prometheus-custom-configure.md)
	    * [prometheus-operator](./prometheus/prometheus-operator/prometheus-operator.md)
	    * [prometheus-operator部署prometheus](./prometheus/prometheus-operator/prometheus-operator.md)
	    * [自定义配置部署prometheus](./prometheus/prometheus-operator/prometheus-custom-configure.md)
	    * [prometheus单实例部署](./prometheus/prometheus-sample.md)
	    * [prometheus部署k8s集群监控](./prometheus/prometheus-k8s.md)
	    * [kube-state-metrics](./prometheus/prometheus—kube-state-metrics.md)
	    * [prometheus_hostpath](./prometheus/prometheus_deploy_hostpath.md)
	    * [自己写的钉钉python消息转发服务](./prometheus/prometheus_dingtalk_pythonproxy.md)
	    * [部署钉钉一个go写的消息转发服务](./prometheus/deploy_dingtalk_proxy.md)
	  * [部署helm](./helm/install_helm.md)
	  * [jenkins_deploy](./jenkins_deploy/jenkins_deploy_index.md)
	    * [创建jenkins命名空间](./jenkins_deploy/jenkins-namespace.yaml.md)
	    * [创建jenkins调用k8s的rbac](./jenkins_deploy/jenkins-rbac.yaml.md)
	    * [创建jenkins-pv](./jenkins_deploy/jenkins_pv.yaml.md)
	    * [创建jenkins-pvc](./jenkins_deploy/jenkins_pvc.yaml.md)
	    * [创建jenkins部署集](./jenkins_deploy/jenkins_deploy_pvc.yaml.md)
	    * [创建jenkins service](./jenkins_deploy/jenkins_web_svc.yaml.md)
	    * [创建jenkins 用于slave的service](./jenkins_deploy/jenkins_slave_svc.yaml.md)
	    * [创建jenkins的ingress](./jenkins_deploy/jenkins-ingress.yaml.md)
	  * [coredns部署](./coredns.md)
	  * [ops](./ops/index.md)
	    * [使用kube-router代理calico发布serviceip以及内网dns配置中继](./ops/kube-router-use-firewall-use-proxy-and-provi-service-ip.md)
	    * [etcd集群备份和恢复](./ops/etcd_cluster_backup_recovery.md)
	    * [初始化节点脚本](./ops/init_nodes.md)
	    * [创建devuser的开发使用的账户](./创建开发使用的devuser账户.md)
	    * [rbac的说明](./rbac/rbac简介.md)
	    * [整体架构](./ops/k8s整体架构.md)
	    * [pod服务监控](./ops/pod服务的监控.md)
	    * [k8sv1.10.0安装文档](./quickstart/README.md)
	    * [prometheus使用nfs](./prometheus/prometheus-operator/prometheus_use_nfs.md)
	    * [alertmanager使用nfs](./prometheus/prometheus-operator/alertmanager-use-nfs.md)
	  * [部署mysqlmgr](./mysql_mgr_deploy/index.md)
	    * [部署mysqlmgr主](./mysql_mgr_deploy/mysql-mgr-0.md)
	    * [部署mysqlmgr从](./mysql_mgr_deploy/mysql-mgr-1.md)
# k8s-install-note
# k8s-install-note
# k8s-install-note
