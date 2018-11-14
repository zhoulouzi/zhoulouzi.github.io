---
title: "minikube install offline"
date: 2017-03-07T12:00:00+08:00
tags:
- minkube
archives:
- kubernetes
categories:
- kubernetes
coverImage: "https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355392/ChMkJ1f8ljWIBAmcAA-gWT6p-0oAAWzegGSHVwAD6Bx012_telyks.jpg"
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1542165911/favicon_z3wusk.png
---

Install minikube offline.

<!--more-->

# minikube install offline step by step

### 目标：

在没有网络接入的情况下安装minikube。供公司app demo 演示使用环境,建议在网络正常的情况下使用一次minikube，然后在尝试offline的安装。

### 准备
需要提前下载几个东西：
 
 1. kubectl 的二进制文件 官网下载 放到/usr/local/bin/ 下即可 
 2. minikube 的二进制文件 官网下载放到/usr/local/bin/ 下即可 
 3. docker的离线安装包 docker 离线安装 
 4. minikube要跑起来所需要的docker镜像：
 
		gcr.io/google_containers/kubernetes-dashboard-amd64 v1.6.3
		gcr.io/google_containers/k8s-dns-sidecar-amd64 1.14.5
		gcr.io/google_containers/k8s-dns-kube-dns-amd64 1.14.5
		gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64 1.14.5
		gcr.io/google-containers/kube-addon-manager v6.4-beta.2
		gcr.io/google_containers/pause-amd64 3.0
		docker image save 导出tar包，方便随时在离线环境使用
5. minikue.iso
	下载地址：[minikube.iso](https://storage.googleapis.com/minikube/iso/minikube-v0.23.5.iso)
	
### 使用定制参数启动minikube
	/usr/local/bin/minikube start --vm-driver=none --iso-url file://tmp/minikube-v0.23.5.iso --kubernetes-version v1.7.5 --extra-config=apiserver.Service.NodePortRange=0-60000

### 完成
