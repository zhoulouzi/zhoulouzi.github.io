---
title: "Configure the Kubernetes Aggregation Layer"
date: 2018-12-07T21:48:15+08:00
tags:
- kubernetes
archives:
- kubernetes
categories:
- kubernetes
coverImage: "https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355392/ChMkJ1f8ljWIBAmcAA-gWT6p-0oAAWzegGSHVwAD6Bx012_telyks.jpg"
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1542165911/favicon_z3wusk.png
---
How to Configure the Kubernetes Aggregation Layer
<!--more-->
### What
Aggregation layer 和 CustomResourceDefinition 都是 Kubernetes为扩展API的方式，两者的对比:[Comparing Aggregation and CRD](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#comparing-ease-of-use)

###
kubernetes最新的一个系统监控组件metrics-server(用来代替heapster)在部署的过程中需要api-server提前配置好 Aggregation Layer，但是配置相关的官方文档很简短,并没有详细的步骤,现在这里介绍下怎么配置,证书生成和其他一些细节.

### Configure the Kubernetes Aggregation Layer
在kubernetes核心代码里有一个组件 kube-aggregator,实现了以下3个功能：
- Provide an API for registering API servers.
- Summarize discovery information from all the servers.
- Proxy client requests to individual servers.

最后可能的request path如下
```    
    client ----> proxy(kube-aggregator) ----> kube-apiserver
                                        ----> (ex. metrics-server)
```
#### 生成生证书
proxy-client certs就是提供与后端apiserver双向TLS验证过程中的凭证,其中证书的CN属性 需要和apiserver的--requestheader-allowed-names值一致.

Install cfssl.

    [cfssl](https://github.com/cloudflare/cfssl)

##### 生成 Kubernetes-front-proxy CA 证书
kubernetes-front-proxy-ca-csr.json:
```
{
    "CN": "Kubernetes-front-proxy CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BJ",
            "ST": "BJ",
            "O": "Kubernetes-front-proxy CA",
            "OU": "Kubernetes-front-proxy CA"
        }
    ]
}
```
使用下面的证书生成CA的证书
```
$ cfssl gencert -initca ./kubernetes-front-proxy-CA-csr.json | cfssljson -bare kubernetes-front-proxy-ca
```
##### 生成 front-proxy-client 证书
ca-config.json:
```
{
    "signing": {
        "default": {
            "expiry": "876000h"
        },
        "profiles": {
            "kubernetes-front-proxy": {
                "expiry": "876000h",
                "usages": [
                    "signing",
                    "key encipherment",
		            "client auth"
                ]
            }
        }
    }
}
```
kubernetes-front-proxy.csr:
```
{
  "CN": "front-proxy-client",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
        {
            "C": "CN",
            "L": "BJ",
            "ST": "BJ",
            "O": "front-proxy-client",
            "OU": "front-proxy-client"
        }
    ]
}
```
使用以下命令生成 front-proxy-client 证书：
```
$ cfssl gencert -ca=kubernetes-front-proxy-ca.pem -ca-key=kubernetes-front-proxy-ca-key.pem  --config=ca-config.json -profile=kubernetes-front-proxy ./kubernetes-front-proxy.csr | cfssljson -bare kubernetes-front-proxy
```


#### kube-apiserver 添加启动参数
``` 
 --requestheader-client-ca-file=<path to aggregator CA cert>
 --requestheader-allowed-names=front-proxy-client
 --requestheader-extra-headers-prefix=X-Remote-Extra-
 --requestheader-group-headers=X-Remote-Group
 --requestheader-username-headers=X-Remote-User
 --proxy-client-cert-file=<path to aggregator proxy cert>
 --proxy-client-key-file=<path to aggregator proxy key>
```
Tips:如果你的api-server节点上没有kube-proxy,额外添加参数
```
--enable-aggregator-routing=true
```
### 参考

- [configure-aggregation-layer](https://kubernetes.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer/)
- [aggregated-api-servers](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/aggregated-api-servers.md)
- [Two Ways to Extend the K8s API](https://docs.google.com/document/d/1y16jKL2hMjQO0trYBJJSczPAWj8vAgNFrdTZeCincmI/edit#)


