---
title: "Use cert-manager to automate ingress's tls with letsencrypt."
date: 2018-04-07T21:48:15+08:00
tags:
- kubernetes
archives:
- kubernetes
categories:
- kubernetes
coverImage: "https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355392/ChMkJ1f8ljWIBAmcAA-gWT6p-0oAAWzegGSHVwAD6Bx012_telyks.jpg"
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1542165911/favicon_z3wusk.png
---
Use cert-manager to automate ingress's tls with letsencrypt.
<!--more-->
### What

cert-manager is a Kubernetes add-on to automate the management and issuance of TLS certificates from various issuing sources.
It will ensure certificates are valid and up to date periodically, and attempt to renew certificates at an appropriate time before expiry.

### Steps

我们将做一个自动为kubernetes-dashoboard申请letsencrypt证书并且暴露在ingress入口上的例子.

> Requirement:

- cloudflare account api-key for ACME dns01 check
- cert-manager installed
- ingress-nginx installed
- kubernetes-dashboard installed
-  安装可以参考 [addon configs](https://github.com/zhoulouzi/easy-kubernetes-HA-cluster/tree/master/addon_config)


1. 把 cloudflare-api-key 做成 kubernetes secret
```
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-key
  namespace: kube-system
data:
   # echo -n "password" | base64
   api: xxx
```

2. 定义 ClusterIssuer for cert-manager
```
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: user@example.com

    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod

    # ACME DNS-01 provider configurations
    dns01:

      # Here we define a list of DNS-01 providers that can solve DNS challenges
      providers:

        - name: cf-dns
          cloudflare:
            email: user@example.com
            # A secretKeyRef to a cloudflare api key
            apiKeySecretRef:
              name: cloudflare-api-key
              key: api
```

3. 定义 kubernetes dashboard的 ingress

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubernetes-dashboard
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    certmanager.k8s.io/cluster-issuer: letsencrypt-prod
    certmanager.k8s.io/acme-challenge-type: dns01
spec:
  tls:
  - hosts:
    - kubernetes-dashboard.example.com
    secretName: kubernetes-dashboard
  rules:
  - host: kubernetes-dashboard.example.com
    http:
      paths:
      - backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```

最后一步提交给kube-apiserver之后等待 DNS01 check通过.
访问https://kubernetes-dashboard.example.com 验证证书.

### 参考

- [offical docs](https://cert-manager.readthedocs.io/en/latest/index.html)


