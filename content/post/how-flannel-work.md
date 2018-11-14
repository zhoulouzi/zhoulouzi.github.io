---
title: "How does flannel work with kubernetes"
date: 2018-02-14T19:00:00+08:00
tags:
- kubernetes
archives:
- kubernetes
categories:
- kubernetes
coverImage: "https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355392/ChMkJ1f8ljWIBAmcAA-gWT6p-0oAAWzegGSHVwAD6Bx012_telyks.jpg"
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1542165911/favicon_z3wusk.png
---

了解flannel作为kubernetes的cni插件是如何工做的。
<!--more-->

在解决容器网络的问题上,docker和kubernetes走了两条路子，分别提出了CNM和CNI两个标准.但是kubernetes逐渐成为容器集群方案首选的时候,作为使用者,我们不得不去调研和测试开源的cni-plugin来解决我们容器环境的网络问题.cluster network addons来解决集群里网络互联互通的问题.kubernetes在[kubernetes networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) 明确提出对CNI插件实现的要求：
1. all containers can communicate with all other containers without NAT
2. all nodes can communicate with all containers (and vice-versa) without NAT
3. the IP that a container sees itself as is the same IP that others see it as

官网列出了很多实践,除了云平台提供的实现外,比较活跃的项目有calico,flannel,weave.每个project各有优点我们来看一个早期的对比文章[calico-flannel-weave](http://chunqi.li/2015/11/15/Battlefield-Calico-Flannel-Weave-and-Docker-Overlay-Network/).除此之外，还建议看看docker官方CNM的文档[docker-CNM](https://success.docker.com/article/networking)
对比理解.


# Flannel
## 安装
>   推荐参考kubenretes和flannel的文档，不多赘述.

## flannel的工作原理:
>   flannel is a virtual network that gives a subnet to each host for use with container runtimes.

目前支持两种backing store来存储配置信息.
1. etcd
2. kubernetes API(kube subnet manager)

Recommended backends
1. VXLAN      vxlan是官方推荐的模式,注意需要kernel支持.
2. host-gw    openshift选择的模式.
3. UDP        debugging only or for very old kernels that don't support VXLAN.

Experimental backends
1. AWS VPC
2. Alloc
3. AliVPC
4. IPSec
5. IPIP
6. GCE

## backend 实现:

### host-gw
> Use host-gw to create IP routes to subnets via remote machine IPs. Requires direct layer2 connectivity between hosts running flannel.

!(host-gw)[https://photos.app.goo.gl/8SYTGDj6C46UY6GZA]

host-gw模式下每个flanneld进程来将backing store里的节点信息和子网信息直接设置到host的路由表上，通过图表我们可以很直观的看到，host1上的pod1发送到host2上的pod2的packets没有经过任何封装(比如vxlan)，host-gw的实现方式和calico一样是一个layer3的实现，由于没有封装和解封所以性能损耗很小.Ps：那这种模式带来的限制就是节点直接需要layer2直连. 

参考:
[](https://github.com/coreos/flannel/blob/master/Documentation/backends.md)
[](https://docs.openshift.com/container-platform/3.4/architecture/additional_concepts/flannel.html)

### vxlan
> Use in-kernel VXLAN to encapsulate the packets.

关于vxlan的知识网络上是非常多的，简单来说就是在现有的layer3网络上实现了一套layer2网络的协议.

参考:
[](https://github.com/coreos/flannel/blob/master/Documentation/backends.md)
### udp
UDP模式下使用flanneld这个进程来封装packets.
![udp](https://photos.app.goo.gl/DVVqiSUhQSFrs96o7)
这种模式增加了packets在user space和kernel space 来回copy的消耗,所以性能很差,官方只推荐在debug的时候来使用.
![user-space](https://photos.app.goo.gl/FndVYFRxses2MaaD9)
参考:
[flannel-backend](https://github.com/coreos/flannel/blob/master/Documentation/backends.md)
[kubernetes-flannel-networking](https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c)
[using-coreos-flannel-for-docker-networking](https://www.slideshare.net/lorispack/using-coreos-flannel-for-docker-networking)