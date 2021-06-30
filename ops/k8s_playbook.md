**1.k8s node节点的管理**

将k8s-node1节点设置为不可调度模式

`kubectl cordon k8s-node1`   

#将当前运行在k8s-node1节点上的容器驱离

`kubectl drain k8s-node1`    #--ignore-daemonsets 忽略daemonset的部署。  

#执行完维护后，将节点重新加入调度

`kubectl uncordon k8s-node1`  
