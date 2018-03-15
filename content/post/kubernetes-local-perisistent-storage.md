---
title: "kubernetes-local-perisistent-storage"
date: 2018-03-07T21:48:15+08:00
draft: false
slug: "kubernetes-local"
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


在公司kubernetes集群是否使用ceph、gluster、nfs等 Dynamic volume provisioning 的问题上，对性能和成本方面的考虑没有使用。但是kuberntes volume 本身提供的hostpath对于statefulset的服务非常不方便使用，特别是volume的挂载和pod 的调度上（之前我们是利用了hostpath和nodeselector做的），在详细的看过kubernets文档之后，我们把精力都聚焦在了 volume ”local“ 这个功能上：
    首先，我们来看一下文档关于local volume的介绍（ https://kubernetes.io/docs/concepts/storage/volumes/#local ）：

        local
        FEATURE STATE: Kubernetes v1.7 alpha

        This alpha feature requires the PersistentLocalVolumes feature gate to be enabled.

        Note: Starting in 1.9, the VolumeScheduling feature gate must also be enabled.

        A local volume represents a mounted local storage device such as a disk, partition or directory.

        Local volumes can only be used as a statically created PersistentVolume.

        Compared to hostPath volumes, local volumes can be used in a durable manner without manually scheduling pods to nodes, as the system is aware of the volume’s node constraints by looking at the node affinity on the PersistentVolume.

        However, local volumes are still subject to the availability of the underlying node and are not suitable for all applications

        Starting in 1.9, local volume binding can be delayed until pod scheduling by creating a StorageClass with volumeBindingMode set to WaitForFirstConsumer. See the example. Delaying volume binding ensures that the volume binding decision will also be evaluated with any other node constraints the pod may have, such as node resource requirements, node selectors, pod affinity, and pod anti-affinity.

        For details on the local volume type, see the Local Persistent Storage user guide.


    最后给了一个github的地址：https://github.com/kubernetes-incubator/external-storage/tree/master/local-volume

  下面就总结下，我利用这个手册做的一些操作。
  环境： kubernets 1.9.3

  step1：
     api-server, controller-manager, scheduler, and all kubelets 开启 feature-gates的功能
     flag：
     --feature-gates=PersistentLocalVolumes=true,VolumeScheduling=true,MountPropagation=true

  step2:
     Creating a StorageClass:
     $ cat local-storage.yaml
          # Only create this for K8s 1.9+
          apiVersion: storage.k8s.io/v1
          kind: StorageClass
          metadata:
            name: local-storage
          provisioner: kubernetes.io/no-provisioner
          volumeBindingMode: WaitForFirstConsumer


  step3:
     Manually create local persistent volume
     #文档中提供了一个 local volume static provisioner ，大概的功能就是 自动将你定义的path里的子文件夹 创建成 persistent volume
     #这里没有使用这种方式，而是选择了手动创建。 这里一定要关注一下 pv的 accessModes persistentVolumeReclaimPolicy 这两个参数，理解他们的意思。
     $cat local-persistent-volume.yaml
          apiVersion: v1
          kind: PersistentVolume
          metadata:
            name: example-local-pv
            annotations:
              "volume.alpha.kubernetes.io/node-affinity": '{
                "requiredDuringSchedulingIgnoredDuringExecution": {
                  "nodeSelectorTerms": [
                    { "matchExpressions": [
                      { "key": "kubernetes.io/hostname",
                        "operator": "In",
                        "values": ["192.168.199.140"]
                      }
                    ]}
                   ]}
                  }'
          spec:
            capacity:
              storage: 5Gi
            accessModes:
            - ReadWriteOnce
            persistentVolumeReclaimPolicy: Retain
            storageClassName: local-storage
            local:
              path: /local_volume

  step4:
      创建一个 statefulset 服务（当你需要replicas为多个时，需要提前创建多个pv在local-storageclass里）：
      $cat local-statefulset.yaml
          apiVersion: apps/v1beta1
          kind: StatefulSet
          metadata:
            name: local-test
          spec:
            serviceName: "local-service"
            replicas: 1
            template:
              metadata:
                labels:
                  app: local-test
              spec:
                containers:
                - name: test-container
                  image: gcr.io/google_containers/busybox:1.24
                  command:
                  - "/bin/sh"
                  args:
                  - "-c"
                  - "count=0; count_file=\"/usr/test-pod/count\"; test_file=\"/usr/test-pod/test_file\"; if [ -e $count_file ]; then count=$(cat $count_file); fi; echo $((count+1)) > $count_file; while [ 1 ]; do date >> $test_file; echo \"This is $MY_POD_NAME, count=$(cat $count_file)\" >> $test_file; sleep 10; done"
                  volumeMounts:
                  - name: local-vol
                    mountPath: /usr/test-pod
                  env:
                  - name: MY_POD_NAME
                    valueFrom:
                      fieldRef:
                        fieldPath: metadata.name
                securityContext:
                  fsGroup: 1234
            volumeClaimTemplates:
            - metadata:
                name: local-vol
              spec:
                accessModes: [ "ReadWriteOnce" ]
                storageClassName: "local-storage"
                resources:
          requests:
            storage: 1Gi


  另外你可以选择不同 local-storage class name 来为不同的服务提供volume。


