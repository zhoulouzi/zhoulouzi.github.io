---
title: "minikube offline installed"
date: 2017-10-07T21:48:15+08:00
draft: false
slug: "minikube"
tags:
- minkube
archives:
- kubernetes
categories:
- docker
- kubernetes
clearReading: true
thumbnailImage: cover.jpg
thumbnailImagePosition: bottom
autoThumbnailImage: yes
metaAlignment: center
comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
---


## 目标：
在没有网络接入的情况下安装minikube。供公司app demo 演示使用环境。

## 建议在网络正常的情况下使用一次minikube，然后在做offline的安装

需要提前下载几个东西：
    1、kubectl 的二进制文件 官网下载 放到/usr/local/bin/ 下即可
    2、minikube 的二进制文件 官网下载 放到/usr/local/bin/ 下即可
    3、docker的离线安装包 docker 离线安装

    4、minikube 要跑起来所需要的docker镜像：
        gcr.io/google_containers/kubernetes-dashboard-amd64    v1.6.3
        gcr.io/google_containers/k8s-dns-sidecar-amd64         1.14.5
        gcr.io/google_containers/k8s-dns-kube-dns-amd64        1.14.5
        gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64   1.14.5
        gcr.io/google-containers/kube-addon-manager            v6.4-beta.2
        gcr.io/google_containers/pause-amd64                   3.0

        docker image save 到一个tar包即可，同时也可以将所需的app docker镜像 save下来

        minikube addons 里面disabled 的 镜像未做研究。

    5、minikue.iso   下载地址：https://storage.googleapis.com/minikube/iso/minikube-v0.23.5.iso


minikube 启动参数：

/usr/local/bin/minikube start --vm-driver=none --iso-url file://tmp/minikube-v0.23.5.iso --kubernetes-version v1.7.5 --extra-config=apiserver.Service.NodePortRange=0-60000

确保 一个localkube的服务正常启动

做完这些 正常使用kubectl apply 你的服务。
