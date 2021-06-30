``` bash
#!/bin/sh

# First Boot of init

timestamp() {
    date +'%Y-%m-%d %H:%M:%S'
}
die (){
    ts=`timestamp`
    echo "$ts ERR: $@"
    exit 1
}
log (){
    ts=`timestamp`
    echo "$ts INFO: $@"
}
warn (){
    ts=`timestamp`
    echo "$ts WARN: $@"
}
run() {
    log "$@"
    $@
}
get_ip() {
    #L_IP=`/usr/sbin/ifconfig | grep -P -o "(?<=inet)(.*)(?=netmask)" | grep -v "127.0.0.1" `
    L_IP=`hostname -I| cut -d" " -f 1`
    echo $L_IP
    [ -z "$L_IP" ] && L_IP=null && die "获取本机IP失败!!!"
}

log "===================================="
log "====**   Activating Begin     **===="
log "===================================="
selinux_status=$(/usr/sbin/getenforce)
if [ $selinux_status == "Disabled" ] ;then echo "selinux_status is disabled,continue...";else echo "selinux status is ${selinux_status},install will breakup...";exit 0 ;fi
cd /tmp
get_ip

URL_BASE='http://10.255.57.7:8090/upload/k8s'

MANIFESTS_DIR='/etc/kubernetes/manifests'
SSL_DIR='/etc/kubernetes/ssl'

[ -d "$MANIFESTS_DIR" ]  || run "mkdir -p $MANIFESTS_DIR"
[ -d "$SSL_DIR" ] || run "mkdir -p $SSL_DIR "
run " mkdir -p /etc/cni/net.d  /opt/cni/bin/ /var/lib/kubelet "
if ! grep "hub-dev.example.com" /etc/hosts ; then  echo "10.255.57.7 hub-dev.example.com" >> /etc/hosts; fi
log "Update os software..."
run "yum update -y -q"
if [ $? == 0 ];then echo "Update sucess...";else echo "Update failed,install will breakup...";exit 0 ;fi
rpm -qa |grep -q 'docker-ce'
if [ $? -ne 0 ];then
    rm -rf /tmp/docker_install_pkg*
    log "install docker-ce-17.03"
    run "wget -q --timeout=30 -O /tmp/ $URL_BASE/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm" \
        || die "Download failed on docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm!!!"
    run "wget -q --timeout=30 -O /tmp/ $URL_BASE/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm" \
        || die "Download failed on docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm!!!"
    log "Install docker-ce-17.03..."
    run "yum install /tmp/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm  -y" \
        || die "Install pkg failed on docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm !!!"
    run "yum install /tmp/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm  -y" \
        || die "Install pkg failed on docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm  !!!"
else
   log "Docker-versio docker-ce is already installed"
fi
log "Download docker.service config..."
run "wget -q --timeout=5 -O /usr/lib/systemd/system/docker.service $URL_BASE/docker.service" \
    || die "Download config on docker.service !!!"

log "Download sysctl.conf config..."
run "wget -q --timeout=5 -O /etc/sysctl.conf $URL_BASE/sysctl.conf" \
    || die "Download config on sysctl.conf !!!"

log "Download kubelet pkg and config..."
run "wget -q --timeout=60 -O /usr/local/bin/kubelet $URL_BASE/kubelet" \
    || die "Download pkg on kubelet !!!"
run "chmod +x /usr/local/bin/kubelet"
run "wget -q --timeout=5 -O /etc/kubernetes/bootstrap.kubeconfig $URL_BASE/bootstrap.kubeconfig" \
    || die "Download config on bootstrap.kubeconfig !!!"
run "wget -q --timeout=5 -O /usr/lib/systemd/system/kubelet.service $URL_BASE/kubelet.service" \
    || die "Download config on kubelet.service !!!"
run "sed -i s#xxx.xxx.xxx.xxx#$L_IP#g /usr/lib/systemd/system/kubelet.service" \
    || die "Fix config on kubelet.service !!!"

log "Download kube-proxy.kubeconfig configfile..."
run "wget -q --timeout=5 -O /etc/kubernetes/kube-proxy.kubeconfig $URL_BASE/kube-proxy.kubeconfig" \
    || die "Download config on kube-proxy.kubeconfig !!!"

log "Download ipvsadm pkg..."
run "wget -q --timeout=60 -O /usr/local/bin/ipvsadm $URL_BASE/ipvsadm" \
    || die "Download pkg on ipvsadm  !!!"
run "chmod +x /usr/local/bin/ipvsadm"

log "Start docker server ..."
run "systemctl enable docker"
run "systemctl start docker" \
    || die "Start docker server faild ... !!!"

log "Start kubelet server ..."
run "systemctl enable kubelet"
run "systemctl start kubelet" \
    || die "Start kubelet server faild ... !!!"


log "===================================="
log "====**    Activating End      **===="
log "===================================="
```
