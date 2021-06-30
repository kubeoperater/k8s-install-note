``` bash
yum remove docker docker-common docker-selinux docker-engine -y
yum erase docker.x86_64 container-selinux.noarch
yum install -y yum-utils device-mapper-persistent-data lvm2
yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm  -y
yum install https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm  -y
# yum install docker-ce
# yum list docker-ce.x86_64  --showduplicates | sort -r
```
