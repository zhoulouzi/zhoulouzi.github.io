---
title: "kubernetes installed"
date: 2017-11-07T21:48:15+08:00
draft: false
slug: "kubernetes"
tags:
- kubernetes
archives:
- kubernetes
categories:
- docker
- kubernetes
clearReading: true
thumbnailImage: http://opiq5jspn.bkt.clouddn.com/mountain_bike_2-wallpaper-3840x2400.jpg
thumbnailImagePosition: bottom
autoThumbnailImage: yes
metaAlignment: center
comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
---


概述:
此文档用于在ubuntu16.04上独立安装kubernetes节点
api-server与kubelet、kube-proxy之间通过tls认证交互
control-manager和scheduler通过api-server在本地暴露的127.0.0.1:8080交互

备注：未实现HA模式  ，实现HA模式，官方的文档（https://kubernetes.io/docs/admin/high-availability/）里指明：需要etcd实现集群模式，apiserver是无状态的，在master节点上正常启动，利用云上的lb做负载均衡，感觉dns也行，注意证书问题就可以。，kube-controller-manager，kube-scheduler需要保证同时只有一个实例在work启动加上--leader-elect启动参数。

etcd组件说明：
    port:
        127.0.0.1:2379: listen-client
        127.0.0.1:2380: initial-cluster
kubelet组件说明：
    port:
        4194:       cadvisor-port                      #cadvisor作为kubernetes一个组件集成在kubelet里
        127.0.0.1:10248:    localhost healthz endpoint #
        10250:     Kubelet to server on  listen for HTTP and respond to a simple API (underspec’d currently) to submit a new manifest.
        10255:    The read-only port for the Kubelet to serve on with no authentication/authorization
            # 只读
# 暴露kubelet里的指标 http://192.168.199.142:10255/stats/summary

kube-proxy组件：
    port：
        127.0.0.1:10249:   metrics server to serve on   # metrics server 并未安装待探索
        10256:   health check server port
        代理的其他服务端口

apiserver 组件说明：
    port:
        127.0.0.1:8080:     insecure-port
        6443:           secure-port

API 认证策略（Authentication strategies）：X509 Client Certs、Service Account Tokens
        # https://kubernetes.io/docs/admin/authentication/
API 授权模式（Authorization Mozules）:Node、RBAC
        # https://kubernetes.io/docs/admin/authorization/

kube-controller-manager组件说明：
    port:
        10252:      the controller-manager's http service runs on
kube-scheduler组件说明：
    port:
        10251:          the scheduler's http service runs on

kube-dns组件说明：
    k8s-dns-sidecar：        # daemon that exports metrics and performs healthcheck on DNS systems.
        10054：      metrics
        dnsmasq:            # 集群内部默认的dns服务
        53  tcp/udp
    kube-dns:           # 与apiserver交互
        10053  tcp/udp      #监听来自dnsmasq的 forward请求
    备注：
    如果reslov.conf 里面是127.0.1.1，本地启动的dnsmasq，在容器里会出现解析外网有问题。
# https://github.com/kubernetes/kubernetes/issues/31337
如果你是这种情况 给你的kubelet添加启动参数 --resolv-conf自定义你的resolv.conf文件或者  DISABLE DNSMASQ的方式完成。
https://docs.docker.com/engine/installation/linux/linux-postinstall/#specify-dns-servers-for-docker 这个文章 可以详细看看


组件清单:
    组件介绍官方文档:https://kubernetes.io/docs/concepts/overview/components/
 kubernetes核心组件:
      二进制:                      版本
    kubectl :kubernetes 客户端工具
    kubelet                     Kubernetes v1.8.3
    kube-proxy                  Kubernetes v1.8.3
      容器方式:                     镜像
    etcd                        gcr.io/google_containers/etcd-amd64:3.0.17
    kube-apiserver                  gcr.io/google_containers/kube-apiserver-amd64:v1.8.3
    kube-controller-manager             gcr.io/google_containers/kube-controller-manager-amd64:v1.8.3
kube-scheduler                  gcr.io/google_containers/kube-scheduler-amd64:v1.8.3
kubernetes-addons: addons 手动部署，自动的好像要加label没搞明白
       容器方式:                        镜像
    kube-dns                    gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.7
                            gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.7
                            gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.7
    kube-dns-autoscaler             gcr.io/google_containers/cluster-proportional-autoscaler-amd64:1.0.0
    kube-dashboard                  gcr.io/google_containers/kubernetes-dashboard-init-amd64:v1.0.1
                            gcr.io/google_containers/kubernetes-dashboard-amd64:v1.7.1
    heapster                    gcr.io/google_containers/heapster-amd64:v1.4.0
    calico                      quay.io/calico/node:v2.6.2
                            quay.io/calico/kube-controllers:v1.0.0
                            quay.io/calico/cni:v1.11.0
另外kubernetes所需的pod根容器:
    pause                       gcr.io/google_containers/pause-amd64:3.0


打包结构如下：
kubernetes_install:
    /binary                     # 包含所需组件的二进制文件和docker镜像
        /二进制\镜像如上列表
        /save.sh                #用于本地打包镜像
    /docker_install
        /docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb
        /install                    #docker 安装脚本
    /conf   配置模板
        /mainfests                  #kubelet manifests yaml
            /etcd.yaml
            /kubernetes-apiserver.yaml
            /kubernetes-controller-manager.yaml
            /kubernetes-scheduler.yaml
/kube-addon-manager.yaml            s
        /addons                 #kubernetes addons yaml
            /kubernetes-dashboard.yaml
/dashboard-admin.yaml       #dashboard的权限
/kubernetes-dns.yml
/heapster.yaml
/heapster-rbac.yaml
/dns-horizontal-autoscaler.yaml
/calico.yaml
/calico-rbac.yaml
    /certs                      #存放生成的证书
        /templates              #cfss csr模板s
/apiserver-csr.conf.template
/ca-config.json         #cfssl ca的config文件
/ca-csr.json                #cfssl ca证书的csr文件
/kube-admin-csr.json.template
/kubelet-csr.json.template
/kube-proxy-csr.json
    /scripts
        /kubernetes_install.sh      #节点执行的脚本
        /node_var_template          #节点变量模板
    /INSTALL                    #安装主脚本
    /cfssl_to_kubernetes.sh         #证书生成脚本，被INSTALL调用
    /cluster_var                    #定义集群参数
    /README.md                  #说明


安装步骤：
1.准备环境:
1.1 安装docker:
建议docker版本:17.03.2-ce
环境确认：
net.ipv4.ip_forward = 1
iptables -P FORWARD ACCEPT          大小写敏感
docker  官方关于 ufw ：forward 表 默认drop的说明：
https://docs.docker.com/engine/installation/linux/linux-postinstall/#allow-access-to-the-remote-api-through-a-firewall
kubernetes  kube-proxy 关于这个问题的fix：
https://github.com/kubernetes/kubernetes/pull/52569

如果docker 安装有问题请参阅:https://docs.docker.com/engine/installation/linux/linux-postinstall/
1.2 环境准备:
关闭swap: swapoff -a
安装conntrack包: apt install conntrack
    # kube-proxy的依赖，没有kube-proxy可能起不来


2.处理iptables:
删除docker创建的网桥:
systemctl stop docker
iptables -t nat -F
ip link set docker0 down
ip link delete docker0

# 使用calico 此处废除。
新建 cbr0:
ip link add name cbr0 type bridge
ip link set dev cbr0 mtu 1460
ip addr add 10.0.0.1/16 dev cbr0                    #此处IP为 pod range ip 的第一位
ip link set dev cbr0 up
iptables -t nat -A POSTROUTING ! -d 192.168.199.0/24 -m addrtype ! --dst-type LOCAL -j MASQUERADE
3.处理docker启动参数:
cp  kubernetes_install/conf/systemd/docker.service  /lib/systemd/system/docker.service
## 具体docker参数: --bridge=cbr0 --ip-masq=false --iptables=false  --bridge=node --exec-root=/var/run/docker
#LimitNOFILE=1048576
#脚本处理时，dockerd的参数全部放置在了daemon.json
# 完整的 dockerd 配置手册 https://docs.docker.com/engine/reference/commandline/dockerd/


4.处理kubelet启动参数
cp  kubernetes_install/conf/systemd/kubelet.service  /lib/systemd/system/kubelet.service
## 具体kubelet参数:   --allow-privileged --kubeconfig=/var/lib/kubelet/kubeconfig --pod-manifest-path=/etc/kubernetes/manifests --cluster-dns=10.96.0.10  --cluster-domain=cluster.local  --register-node --hostname-override=192.168.199.142 --node-ip 192.168.199.142  --network-plugin=cni
5.处理kube-proxy启动参数
cp  kubernetes_install/conf/systemd/kube-proxy.service  /lib/systemd/system/kube-proxy.service
## 具体kube-proxy参数:  --proxy-mode=iptables --hostname-override=192.168.199.142 --master=https://192.168.199.142:6443 --kubeconfig=/var/lib/kube-proxy/kubeconfig --proxy-port-range=1-65535 --cluster-cidr=10.0.0.0/16

#脚本内 --proxy-port-range=1-65535 没有打开 默认3000以上端口，看后期需求

修改参数3\4\5完毕之后执行systemctl daemon-reload
重启3\4\5中的服务

6.复制kubernetes-apiserver.yaml到宿主机/etc/kubernetes/manifests
cp  kubernetes_install/conf/manifests/kubernetes-apiserver.yaml /etc/kubernetes/manifests/kubernetes-apiserver.yaml
#  apiserver的启动参数:
kube-apiserver
    - --allow-privileged=true
    - --address=192.168.199.142
    - --service-cluster-ip-range=10.96.0.0/12
    - --etcd-servers=http://127.0.0.1:2379
    - --client-ca-file=/srv/kubernetes/ca.crt
    - --tls-cert-file=/srv/kubernetes/apiserver.crt
    - --tls-private-key-file=/srv/kubernetes/apiserver.key
    - --admission-control=Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota
    - --allow-privileged=true
    - --insecure-bind-address=127.0.0.1
    - --advertise-address=192.168.199.142
    - --authorization-mode=Node,RBAC
    - --service-node-port-range=0-65535


7.复制yaml配置文件
cp  kubernetes_install/conf/manifests/kubernetes-controller-manager.yaml /etc/kubernetes/manifests/kubernetes-controller-manager.yaml
cp  kubernetes_install/conf/manifests/kubernetes-scheduler.yaml /etc/kubernetes/manifests/kubernetes-scheduler.yaml
cp kubernetes_install/conf/addons/* /etc/kubernetes/addons/


8.证书生成:
#脚本使用的cfssl，简单无脑
8.1 CA证书生成：
openssl genrsa -out ca.key 2048
    openssl req -x509 -new -nodes -key ca.key -subj "/CN=${MASTER_IP}" -days 10000 -out ca.crt

#  Distributing Self-Signed CA Certificate
sudo cp ca.crt /usr/local/share/ca-certificates/kubernetes.crt
sudo update-ca-certificates

8.2 apiserver证书生成：
    openssl genrsa -out apiserver.key 2048
修改 kubernetes_install/cert/openssl/apiserver.csr.conf 文件分别将 <MASTER_IP>   <MASTER_CLUSTER_IP>替换
openssl req -new -key apiserver.key -out apiserver.csr -config apiserver.csr.conf
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key \
 -CAcreateserial -out apiserver.crt -days 10000 \
-extensions v3_ext -extfile apiserver.csr.conf

8.3 kubelet证书生成：
openssl genrsa -out kubelet.key 2048
openssl req -new -key kubelet.key -out kubelet-csr.pem -subj  "/CN=system:node:${nodeip}/O=system:nodes"
        # https://kubernetes.io/docs/admin/authorization/rbac/#default-roles-and-role-bindings
openssl x509 -req -in kubelet-csr.pem -CA ca.crt -CAkey ca.key -CAcreateserial -out kubelet.crt -days 10000

8.4 kube-proxy证书生成：
openssl genrsa -out kube-proxy.key 2048
openssl req -new -key kubelet-proxy.key -out kube-proxy-csr.pem -subj  "/CN=system:kube-proxy"
        # https://kubernetes.io/docs/admin/authorization/rbac/#default-roles-and-role-bindings
openssl x509 -req -in kube-proxy-csr.pem -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-proxy.crt -days 10000

8.4 kube-admin证书生成：
openssl genrsa -out kube-admin.key 2048
openssl req -new -key kube-admin.key -out kube-admin-csr.pem -subj  "/CN=kube-admin/O=system:masters"
        # https://kubernetes.io/docs/admin/authorization/rbac/#default-roles-and-role-bindings
openssl x509 -req -in kube-admin-csr.pem -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-admin.crt -days 10000

将所有生成的crt key 复制到 /srv/kubernetes；should separate differnt crt/key

9 kubeconfig文件生成：
    利用kubectl生成: should point generate config file
9.1 kubelet-kubeconfig 文件生成：
kubectl config set-cluster k8s --certificate-authority=/srv/kubernetes/ca.crt --embed-certs=true --server=https://192.168.199.142:6443
kubectl config set-credentials kubelet --client-certificate=/srv/kubernetes/kubelet.crt --client-key=/srv/kubernetes/kubelet.key --embed-certs=true
kubectl config set-context k8s_kubelet --cluster=k8s --user=kubelet
kubectl config use-context k8s_kubelet
mv /root/.kube/config  /var/lib/kubelet/kubeconfig
9.2 kube-proxy-kubeconfig 文件生成：
kubectl config set-cluster k8s --certificate-authority=/srv/kubernetes/ca.crt --embed-certs=true --server=https://192.168.199.142:6443
kubectl config set-credentials system:kube-proxy --client-certificate=/srv/kubernetes/kube-proxy.crt --client-key=/srv/kubernetes/kube-proxy.key --embed-certs=true
kubectl config set-context k8s_kube-proxy --cluster=k8s --user=system:kube-proxy
kubectl config use-context k8s_kube-proxy
mv /root/.kube/config  /var/lib/kube-proxy/kubeconfig
9.3 admin-kubeconfig 文件生成：
kubectl config set-cluster k8s --certificate-authority=/srv/kubernetes/ca.crt --embed-certs=true --server=https://192.168.199.142:6443
kubectl config set-credentials kube-admin --client-certificate=/srv/kubernetes/ kube-admin.crt --client-key=/srv/kubernetes/ kube-admin.key --embed-certs=true
kubectl config set-context k8s_kube-admin --cluster=k8s --user=kube-admin
kubectl config use-context kube-admin
备份你的admin-kubeconfig。



