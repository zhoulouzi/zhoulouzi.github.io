---
title: "calico-flannel-weave"
date: 2018-08-07T11:48:15+08:00
tags:
- kubernetes
archives:
- kubernetes
categories:
- docker
- kubernetes
coverImage: "https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355392/ChMkJ1f8ljWIBAmcAA-gWT6p-0oAAWzegGSHVwAD6Bx012_telyks.jpg"
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355321/Real_gaggav.png
---

Comparison between calico、flannel、weave.

<!--more-->

# calico:
calico提供了多种部署方式，ipip，node-to-node BGP mesh，global/node specific
需要根据你的依赖网络环境来决定如何部署。 
ipip: calico 会在每个node之间配置一个ip tunnel来转发package
node-to-node BGP mesh: 每个节点利用bird建立bgp peer关系，节点通过路由表来转发packag(官方推荐是小于50个节点)

要看的几篇文章：
https://docs.projectcalico.org/v3.1/reference/architecture/
https://docs.projectcalico.org/v3.1/reference/architecture/components
https://docs.projectcalico.org/v3.1/reference/architecture/data-path

https://docs.projectcalico.org/v3.1/reference/private-cloud/l2-interconnect-fabric
https://docs.projectcalico.org/v3.1/reference/private-cloud/l3-interconnect-fabric

简单测试数据
	[ 6] 0.0-10.0 sec 1.07 GBytes 920 Mbits/sec
	[ 5] 0.0-10.0 sec 1.09 GBytes 937 Mbits/sec


#  flannel

vxlan backend: work in kernel space		  
udp backend: work in userspace            docker0 --- flanel0(tun) --- flanneld
https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c
https://docs.openshift.com/container-platform/3.10/architecture/networking/network_plugins.html
vxlan
https://www.slideshare.net/enakai/how-vxlan-works-on-linux
https://events.static.linuxfound.org/sites/events/files/slides/2013-linuxcon.pdf


# WEAVE NET

FastDataPath: kernel space
sleeve: userspace

## 特点
Virtual Ethernet Switch
Weave Net creates a virtual network that connects Docker containers deployed across multiple hosts.
No External Cluster Store Required
weave不需要累死etcd这种外部存储支持，不需要
Operates in Partially Connected Networks
Weave Net routers 彼此 建立TCP链接交换拓扑信息
Fast data path:
uses the Linux kernel’s Open vSwitch datapath module

官网的这篇文档直接解释了 两种模式的不同，如果内核允许会直接使用FDP模式
https://www.weave.works/docs/net/latest/concepts/fastdp-how-it-works/

https://www.weave.works/docs/net/latest/overview/
https://www.weave.works/docs/net/latest/overview/features/
https://www.weave.works/docs/net/latest/concepts/how-it-works/
https://www.weave.works/docs/net/latest/concepts/fastdp-how-it-works/

https://www.weave.works/blog/weave-docker-networking-performance-fast-data-path/
我也分别在线上环境和测试环境简单利用iperf测试过这个值，和官方给出数据比例基本吻合，测试环境是本地物理环境数据可以，云上数值稍差。	
[ 5] 0.0-10.0 sec 1.04 GBytes 894 Mbits/sec
[ 4] 0.0-10.0 sec 1.05 GBytes 895 Mbits/sec

[ 5] 0.0-10.0 sec 814 MBytes 683 Mbits/sec
[ 4] 0.0-10.0 sec 829 MBytes 694 Mbits/sec
