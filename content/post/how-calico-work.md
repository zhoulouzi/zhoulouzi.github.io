---
title: "How does calico work with kubernetes"
date: 2018-02-14T19:00:00+08:00
tags:
- kubernetes
archives:
- kubernetes
categories:
- kubernetes
coverImage: "https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355392/ChMkJ1f8ljWIBAmcAA-gWT6p-0oAAWzegGSHVwAD6Bx012_telyks.jpg"
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1542166411/uuopzhfeswhmr1tptrs4.jpg
---

了解calico作为kubernetes的cni插件是如何工做的。
<!--more-->

在解决容器网络的问题上,docker和kubernetes分别提出了CNM和CNI两个标准.但是kubernetes逐渐成为容器集群方案首选的时候,作为使用者,我们不得不去调研和测试开源的cni-plugin来解决我们容器环境的网络问题.cluster network addons来解决集群里网络互联互通的问题.kubernetes在[kubernetes networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) 明确提出对CNI插件实现的要求：

1. all containers can communicate with all other containers without NAT
2. all nodes can communicate with all containers (and vice-versa) without NAT
3. the IP that a container sees itself as is the same IP that others see it as

官网列出了很多实践,除了云平台提供的实现外,比较活跃的项目有calico,flannel,weave.每个project各有优点我们来看一个早期的对比文章[calico-flannel-weave](http://chunqi.li/2015/11/15/Battlefield-Calico-Flannel-Weave-and-Docker-Overlay-Network/).除此之外，还建议看看docker官方CNM的文档[docker-CNM](https://success.docker.com/article/networking)
对比理解.

# Calico
> Calico creates and manages a flat layer 3 network, assigning each workload a fully routable IP address.

Calico为容器网络提供了一个layer3的实现方式,packets在endpoints之间不需要封装和NAT.

calico arch:
![calico arch](https://res.cloudinary.com/ddvxfzzbe/image/upload/v1543404693/calico-arch-gen-v3.2_t04nry.svg)


要看的几篇文章：
## 安装
> 推荐参考kubernetes和calico的文档，不多赘述.

[install calico on kubernetes](https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/calico)

## calico的工作原理:
>  In the Calico approach, IP packets to or from a workload are routed and firewalled by the Linux routing table and iptables infrastructure on the workload’s host. 

calico通过控制endpoints宿主机上的路由表和iptables来控制endpoints之间相互发送的packets

详见:[The Calico Data Path](https://docs.projectcalico.org/v3.3/reference/architecture/data-path)

### Node-to-Node mesh
这个模式下, 跑在每个节点上BIRD互为BGP peer,共享容器的路由信息。

需要节点之间二层可达.

### IPIP
IPIP模式实现场景是你没办法完全控制节点之间的network,比如在节点之间非layer2直连的情况下,packets在转发之后无法被正确路由到目的节点.
IPIP是一种把一个IP packets封装在另一个IP packets的 IP tunnel协议
最终包在外边一层header的是宿主机和目的宿主机的IP addrs.

参考:[config ipip](https://docs.projectcalico.org/v3.3/usage/configuration/ip-in-ip)

### BGP Route Reflector

Node-to-Node mesh 在大规模部会遇到性能问题

参考：

https://docs.projectcalico.org/v3.1/reference/architecture/
https://docs.projectcalico.org/v3.1/reference/architecture/components
https://docs.projectcalico.org/v3.1/reference/architecture/data-path
https://docs.projectcalico.org/v3.1/reference/private-cloud/l2-interconnect-fabric
https://docs.projectcalico.org/v3.1/reference/private-cloud/l3-interconnect-fabric
