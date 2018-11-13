---
title: "Using kubeclt explain to write kubernetes yaml"
date: 2018-01-31T21:48:15+08:00
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

Yaml references for kubernetes objects.
如何更快,更准确的编写kubernetes yaml配置文件.

<!--more-->

# kubectl explain
刚开始接触kubernetes的时候,会很头痛yaml配置文件怎么写.
原因大概有几个方面,比如对objects不了解.或者了解但是具体的参数和配置还是需要手册来帮助,[kubernetes API references](https://kubernetes.io/docs/reference/kubernetes-api/)提供了特别详细API手册,但是在使用过程中还是觉得不太方便.
不过大家有没有注意到kubectl提供了sub command 可以很方便的描述每个opjects的参数和配置.

首先来看看kubectl help:
```
$kubectl explain -h
    List the fields for supported resources

    This command describes the fields associated with each supported API resource. Fields are identified via a simple
    JSONPath identifier:

    <type>.<fieldName>[.<fieldName>]

    Add the --recursive flag to display all of the fields at once without descriptions. Information about each field is
    retrieved from the server in OpenAPI format.

    Use "kubectl api-resources" for a complete list of supported resources.

    Examples:
    # Get the documentation of the resource and its fields
    kubectl explain pods

    # Get the documentation of a specific field of a resource
    kubectl explain pods.spec.containers

    Options:
        --api-version='': Get different explanations for particular API version
        --recursive=false: Print the fields of fields (Currently only 1 level deep)

    Usage:
    kubectl explain RESOURCE [options]

    Use "kubectl options" for a list of global command-line options (applies to all commands).
```

我们来看看secrets里有哪些参数吧：
```
$kubectl explain secrets

    kubectl explain secrets --recursive

    DESCRIPTION:
     Secret holds secret data of a certain type. The total bytes of the values
     in the Data field must be less than MaxSecretSize bytes.

    FIELDS:
       apiVersion   <string>
       data <map[string]string>
       kind <string>
       metadata <Object>
          annotations   <map[string]string>
          clusterName   <string>
          creationTimestamp <string>
          deletionGracePeriodSeconds    <integer>
          deletionTimestamp <string>
          finalizers    <[]string>
          generateName  <string>
          generation    <integer>
          initializers  <Object>
             pending    <[]Object>
                name    <string>
             result <Object>
                apiVersion  <string>
                code    <integer>
                details <Object>
                   causes   <[]Object>
                      field <string>
                      message   <string>
                      reason    <string>
                   group    <string>
                   kind <string>
                   name <string>
                   retryAfterSeconds    <integer>
                   uid  <string>
                kind    <string>
                message <string>
                metadata    <Object>
                   continue <string>
                   resourceVersion  <string>
                   selfLink <string>
                reason  <string>
                status  <string>
          labels    <map[string]string>
          name  <string>
          namespace <string>
          ownerReferences   <[]Object>
             apiVersion <string>
             blockOwnerDeletion <boolean>
             controller <boolean>
             kind   <string>
             name   <string>
             uid    <string>
          resourceVersion   <string>
          selfLink  <string>
          uid   <string>
       stringData   <map[string]string>
       type <string>
```


有了这个方法是不是感觉爸爸再也不用担心我怎么写配置了.希望能够帮助到你。

# 额外分享一个工具，有兴趣的同学可以看看是什么.

[Registry creds <-> config.json <-> Kubernetes Secret YAML](https://polyverse.github.io/docker-creds-converter/)