---
title: "How does flannel work with kubernetes"
date: 2018-02-14T19:00:00+08:00
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
了解flannel作为kubernetes的cni插件是如何工做的。
<!--more-->
在装完kubernetes master components(control plane)之后,就必须选择cluster network addons来解决集群里网络互联互通的问题,可以选择的有第三方的CNI networking plugin或者kubenet plugin. [kubernetes cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/) 明确提出对CNI插件实现的要求：
1. all containers can communicate with all other containers without NAT
2. all nodes can communicate with all containers (and vice-versa) without NAT
3. the IP that a container sees itself as is the same IP that others see it as

我会在博客里分别梳理一下calico flannel weave 这三个开源项目的原理，今天的这篇来看看flannel。
# 安装
推荐参考kubenretes和flannel的文档，不多赘述.

# Backend
目前支持的backend:
    ## Recommended backends
    VXLAN      vxlan是官方推荐的模式,注意需要kernel支持.
    host-gw    openshift选择的模式.
    UDP        debugging only or for very old kernels that don't support VXLAN.
    ## Experimental backends
    AWS VPC
    Alloc
    AliVPC
    IPSec
    IPIP
    GCE


## host-gw

Use host-gw to create IP routes to subnets via remote machine IPs. Requires direct layer2 connectivity between hosts running flannel.
!()[]
这这个模式下每个主机上跑了一个flanneld进程来将

https://docs.openshift.com/container-platform/3.4/architecture/additional_concepts/flannel.html
https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c
https://github.com/coreos/flannel/blob/master/Documentation/backends.md