---
title: "How does weave net work with kubernetes"
date: 2018-02-14T19:00:00+08:00
tags:
- kubernetes
archives:
- kubernetes
categories:
- kubernetes
coverImage: "https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355392/ChMkJ1f8ljWIBAmcAA-gWT6p-0oAAWzegGSHVwAD6Bx012_telyks.jpg"
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1542166327/5a3da579-43d8-45a4-a315-cc84147574f3weave-logo-512_u4yo3b.png
---
了解weave net作为kubernetes的cni插件是如何工作的。
<!--more-->

# 容器网络
在解决容器网络的问题上,docker和kubernetes分别提出了CNM和CNI两个标准.但是kubernetes逐渐成为容器集群方案首选的时候,作为使用者,我们不得不去调研和测试开源的cni-plugin来解决我们容器环境的网络问题.cluster network addons来解决集群里网络互联互通的问题.kubernetes在[kubernetes networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) 明确提出对CNI插件实现的要求：

1. all containers can communicate with all other containers without NAT
2. all nodes can communicate with all containers (and vice-versa) without NAT
3. the IP that a container sees itself as is the same IP that others see it as

官网列出了很多实践,除了云平台提供的实现外,比较活跃的项目有calico,flannel,weave net.每个project各有优点我们来看一个早期的对比文章[calico-flannel-weave](http://chunqi.li/2015/11/15/Battlefield-Calico-Flannel-Weave-and-Docker-Overlay-Network/).除此之外，还建议看看docker官方CNM的文档[docker-CNM](https://success.docker.com/article/networking)
对比理解.

# Weave Net
## Weave Net 是什么
> 早期有文章描述说，Weave is described as "a giant Ethernet switch to which all the containers are connected"

> [Weave Net官方文档](https://www.weave.works/docs/net/latest/overview/)把整个工作原理是讲的比较清楚的,结合我们在自己环境中的实验,可能会有一个很快的理解和提升.

Weave Net为容器创造了一个虚拟的二层网络,就好像所有的containers都连接在这个交换机上.那是如何做到呢.Weave Net在每个host上跑一个Weave Net routers,成为整个网络中一个peer节点. 各个peer会建立TCP的链接,
Weave Net 利用Gossip协议协议在peer节点之间share网络拓扑的信息.

## 安装
>   推荐参考kubenretes和Weave Net的文档，不多赘述.
确保节点之间以下端口
    TCP 6783 
    UDP 6783/6784

## Weave 工作原理
Weave net 会在每个节点上创建一个bridge(weave)所有的container都会通过veth设备链接到这个网桥上. 然后通过sleeve或者fast datapath来转发由容器发出来的packets

## 两种模式
### Fast Datapath  
> operates entirely in kernel space.

Fast Datapath mode 需要linux kernel 3.12 及以后的版本(open vSwitch datapath (ODP) and VXLAN features),如果不支持的话，weave会fall back到sleeve mode.

![fastdp](https://res.cloudinary.com/ddvxfzzbe/image/upload/v1542191222/weave-net-fdp1-1024x454_wu3bpw.png)
 
### sleeve  
> captured by the kernel and processed by the Weave Net router in user space, forwarded over UDP to weave router peers running on other hosts

通过pcap来截获发往不同节点的packets，Weave Net Router进程把这些packets封装成UDP包发送给目的节点的Router进程.

sleeve 模式类似flannel的udp模式,不同的是UDP可以一次封装多个发往同一地址的packets
![sleeve](https://res.cloudinary.com/ddvxfzzbe/image/upload/v1542191187/weave-net-encap1-1024x459_yg5hjl.png)


参考：

https://www.weave.works/docs/net/latest/concepts/fastdp-how-it-works/
https://www.weave.works/docs/net/latest/overview/
https://www.weave.works/docs/net/latest/overview/features/
https://www.weave.works/docs/net/latest/concepts/how-it-works/
https://www.weave.works/docs/net/latest/concepts/fastdp-how-it-works/