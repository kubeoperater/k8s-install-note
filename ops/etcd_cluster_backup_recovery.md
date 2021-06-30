# 参考文档

[参考文档 etcd cluster 备份恢复](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/recovery.md)

[etcd性能测试](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/performance.md)


#### etcd 集群失败容忍表

It is recommended to have an odd number of members in a cluster. Having an odd cluster size doesn't change the number needed for majority, but you gain a higher tolerance for failure by adding the extra member. You can see this in practice when comparing even and odd sized clusters:

| Cluster Size | Majority   | Failure Tolerance |
|--------------|------------|-------------------|
| 1 | 1 | 0 |
| 2 | 2 | 0 |
| 3 | 2 | **1** |
| 4 | 3 | 1 |
| 5 | 3 | **2** |
| 6 | 4 | 2 |
| 7 | 4 | **3** |
| 8 | 5 | 3 |
| 9 | 5 | **4** |


首先模拟了etcd集群down机的情况，当集群为3台时，down机一台,在使用命令查询和kubernetes dashboard 查询都不受影响，pod访问正常，down机2台，则访问不到。建议 etcd 集群的台数为5台。


**1.etcd Cluster 备份**

>集群备份：在主机 10.255.72.189上执行：

``` bash
ETCDCTL_API=3 etcdctl \
--cacert=/etc/etcd/ssl/ca.pem \
--cert=/etc/etcd/ssl/etcd.pem \
--key=/etc/etcd/ssl/etcd-key.pem \
--endpoints=https://10.255.72.189:2379,https://10.255.72.190:2379,https://10.255.72.191:2379 snapshot save snapshot.db
```

**2.etcd Cluster 恢复**

> 集群恢复：

#恢复etcd01

``` bash
service etcd stop
mv  /var/lib/etcd/ /var/lib/etcd_bak
rm -fr /var/lib/etcd/
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --name etcd01 --initial-cluster etcd01=https://10.255.72.189:2380,etcd02=https://10.255.72.190:2380,etcd03=https://10.255.72.191:2380  --initial-cluster-token etcd-cluster-1 --initial-advertise-peer-urls https://10.255.73.196:2380 --data-dir=/var/lib/etcd
service etcd start
```
