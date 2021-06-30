
**1.部署一个mysql-mgr-1实例**

#给对应的节点打标签

```
kubectl label node kube-master03 mysqlrole=mysql-mgr-1
```

#mysql-mgr-cnf-1.yaml

这里有一个地方需要注意：因为mysql mgr 会默认使用hostname 做为member_host 所以需要增加变量使mysql mgr采用自定义的域名做member_host mysql的参数为 report_host

[report_host官方解释](https://dev.mysql.com/doc/refman/5.7/en/replication-options-slave.html#option_mysqld_report-host)


``` yaml
apiVersion: v1
data:
  mysql-mgr-1.cnf: |
    [mysqld]
    port = 3306
    character_set_server = utf8
    socket = /tmp/mysql.sock
    basedir = /usr/local/mysql
    log-error = /data/mysql/data/mysql.err
    pid-file = /data/mysql/data/mysql.pid
    datadir = /data/mysql/data
    server_id = 092231
    log_bin = mysql-bin
    relay-log = relay-bin
    back_log = 500
    max_connections = 3000
    wait_timeout = 5022397
    interactive_timeout = 5022397
    max_connect_errors = 1000
    relay-log-recovery=1
    max_allowed_packet = 32M
    sort_buffer_size = 4M
    read_buffer_size = 4M
    join_buffer_size = 8M
    thread_cache_size = 64
    tmp_table_size = 256M
    log_slave_updates=1
    long_query_time = 1
    slow_query_log = 1
    slow_query_log_file = /data/mysql/data/slow_sql.log
    skip-name-resolve
    sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
    innodb_buffer_pool_size=700M
    innodb_data_file_path = ibdata1:1024M:autoextend
    innodb_flush_log_at_trx_commit=1
    innodb_log_buffer_size = 16M
    innodb_log_file_size = 256M
    innodb_log_files_in_group = 2
    innodb_max_dirty_pages_pct = 50
    sync_binlog=1
    master_info_repository=TABLE
    relay_log_info_repository=TABLE
    log_timestamps=SYSTEM
    gtid_mode = ON
    enforce_gtid_consistency = ON
    master_info_repository = TABLE
    relay_log_info_repository = TABLE
    log_slave_updates = ON
    binlog_checksum = NONE
    log_slave_updates = ON
    slave_parallel_type=LOGICAL_CLOCK
    slave_parallel_workers=8
    slave-preserve-commit-order=on
    group_replication_compression_threshold=200000
    transaction_write_set_extraction = XXHASH64
    loose-group_replication_group_name="01e5fb97-be64-41f7-bafd-3afc7a6ab555"
    loose-group_replication_start_on_boot=off
    loose-group_replication_local_address="mysql-mgr-1.mgrtest.mysqldb.svc.cluster.local.:13306"
    loose-group_replication_group_seeds="mysql-mgr-0.mgrtest.mysqldb.svc.cluster.local.:13306,mysql-mgr-1.mgrtest.mysqldb.svc.cluster.local.:13306,mysql-mgr-2.mgrtest.mysqldb.svc.cluster.local.:13306"
    loose-group_replication_bootstrap_group = off
    loose-group_replication_ip_whitelist='10.255.0.0/24,172.17.0.0/24,10.229.0.0/16,10.228.0.0/16'
    [mysqldump]
    quick
    max_allowed_packet = 32M
kind: ConfigMap
metadata:
  name: mysql-mgr-1-cnf
  namespace: mysqldb
```

# mysql-mgr-0-pod.yaml

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-mgr-1
  namespace: mysqldb
  labels:
    name: mysql-mgr
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: mysqlrole
              operator: In
              values: ["mysql-mgr-1"]
  hostname: mysql-mgr-1
  subdomain: mgrtest
  containers:
  - image: hub-dev.example.com/base/mysqlmgr:v0.1.8
    name: mysql-mgr-1
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: tz-config
      mountPath: /etc/localtime
    - name: mysql-data
      mountPath: /data/mysql/data/
    - name: mysql-config
      mountPath: /etc/my.cnf
      subPath: my.cnf
    env:
    - name: INNODB_BUFFER_POOL_SIZE
      value: 500M
  volumes:
    - name: tz-config
      hostPath:
        path: /etc/localtime
    - name: mysql-data
      hostPath:
        path: /data/mysql/data/
    - name: mysql-config
      configMap:
        name: mysql-mgr-1-cnf
        items:
          - key: mysql-mgr-1.cnf
            path: my.cnf
```
