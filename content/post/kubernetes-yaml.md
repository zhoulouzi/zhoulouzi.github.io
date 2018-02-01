---
title: "kubernetes yaml 配置文件的 reference"
date: 2017-11-07T21:48:15+08:00
draft: false
slug: "kubernetes"
tags:
- kubernetes
archives:
- kubernetes
categories:
- docker
- kubernetes
clearReading: true
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355321/Real_gaggav.png
thumbnailImagePosition: right
autoThumbnailImage: yes
metaAlignment: center
comments: true
showTags: true
showPagination: true
showSocial: true
showDate: true
---

## 怎么找到编写kubernetes yaml 配置文件的手册

    其实如果你装好了了kubectl你就随时随地的可以找到配置文件怎么写。

    # kubectl explain -h

    kubectl explain secrets

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

    试着在终端敲下这几个命令。（爸爸再也不用担心我写配置了）



## 额外分享一个工具。

    Registry creds <-> config.json <-> Kubernetes Secret YAML

    https://polyverse.github.io/docker-creds-converter/