---
title: "docker pre-install kernel-check"
date: 2017-03-07T11:48:15+08:00
tags:
- docker
archives:
- docker
categories:
- docker
coverImage: "https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355392/ChMkJ1f8ljWIBAmcAA-gWT6p-0oAAWzegGSHVwAD6Bx012_telyks.jpg"
thumbnailImage: https://res.cloudinary.com/ddvxfzzbe/image/upload/v1513355321/Real_gaggav.png
---
pre-check before install docker

<!--more-->

# docker pre-install kernel-check

Docker官方明确要求了kernel版本要在version 3.10及以上，（但是在centos上还是遇到了很多bug，如果你用centos7我觉得应该在3.18以上，ubuntu的内核比较新，出问题较少）另外官方还是提供了一个check脚本，所以我们可以利用这个脚本check下我们的kernel缺少那些东西。

## step

    1、curl https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh > check-config.sh
    2、bash ./check-config.sh
## result
    [root@master1 ~]# bash check-config.sh
    warning: /proc/config.gz does not exist, searching other paths for kernel config ...
    info: reading kernel config from /boot/config-3.10.0-693.el7.x86_64 ...

    Generally Necessary:
    - cgroup hierarchy: properly mounted [/sys/fs/cgroup]
    - CONFIG_NAMESPACES: enabled
    - CONFIG_NET_NS: enabled
    - CONFIG_PID_NS: enabled
    - CONFIG_IPC_NS: enabled
    - CONFIG_UTS_NS: enabled
    - CONFIG_CGROUPS: enabled
    - CONFIG_CGROUP_CPUACCT: enabled
    - CONFIG_CGROUP_DEVICE: enabled
    - CONFIG_CGROUP_FREEZER: enabled
    - CONFIG_CGROUP_SCHED: enabled
    - CONFIG_CPUSETS: enabled
    - CONFIG_MEMCG: enabled
    - CONFIG_KEYS: enabled
    - CONFIG_VETH: enabled (as module)
    - CONFIG_BRIDGE: enabled (as module)
    - CONFIG_BRIDGE_NETFILTER: enabled (as module)
    - CONFIG_NF_NAT_IPV4: enabled (as module)
    - CONFIG_IP_NF_FILTER: enabled (as module)
    - CONFIG_IP_NF_TARGET_MASQUERADE: enabled (as module)
    - CONFIG_NETFILTER_XT_MATCH_ADDRTYPE: enabled (as module)
    - CONFIG_NETFILTER_XT_MATCH_CONNTRACK: enabled (as module)
    - CONFIG_NETFILTER_XT_MATCH_IPVS: enabled (as module)
    - CONFIG_IP_NF_NAT: enabled (as module)
    - CONFIG_NF_NAT: enabled (as module)
    - CONFIG_NF_NAT_NEEDED: enabled
    - CONFIG_POSIX_MQUEUE: enabled
    - CONFIG_DEVPTS_MULTIPLE_INSTANCES: enabled

    Optional Features:
    - CONFIG_USER_NS: enabled
      (RHEL7/CentOS7: User namespaces disabled; add 'user_namespace.enable=1' to boot command line)
    - CONFIG_SECCOMP: enabled
    - CONFIG_CGROUP_PIDS: enabled
    - CONFIG_MEMCG_SWAP: enabled
    - CONFIG_MEMCG_SWAP_ENABLED: enabled
        (cgroup swap accounting is currently enabled)
    - CONFIG_MEMCG_KMEM: enabled
    - CONFIG_RESOURCE_COUNTERS: missing
    - CONFIG_BLK_CGROUP: enabled
    - CONFIG_BLK_DEV_THROTTLING: enabled
    - CONFIG_IOSCHED_CFQ: enabled
    - CONFIG_CFQ_GROUP_IOSCHED: enabled
    - CONFIG_CGROUP_PERF: enabled
    - CONFIG_CGROUP_HUGETLB: enabled
    - CONFIG_NET_CLS_CGROUP: enabled
    - CONFIG_NETPRIO_CGROUP: enabled
    - CONFIG_CFS_BANDWIDTH: enabled
    - CONFIG_FAIR_GROUP_SCHED: enabled
    - CONFIG_RT_GROUP_SCHED: enabled
    - CONFIG_IP_VS: enabled (as module)
    - CONFIG_IP_VS_NFCT: enabled
    - CONFIG_IP_VS_RR: enabled (as module)
    - CONFIG_EXT3_FS: missing
    - CONFIG_EXT3_FS_XATTR: missing
    - CONFIG_EXT3_FS_POSIX_ACL: missing
    - CONFIG_EXT3_FS_SECURITY: missing
        (enable these ext3 configs if you are using ext3 as backing filesystem)
    - CONFIG_EXT4_FS: enabled (as module)
    - CONFIG_EXT4_FS_POSIX_ACL: enabled
    - CONFIG_EXT4_FS_SECURITY: enabled
    - Network Drivers:
      - "overlay":
        - CONFIG_VXLAN: enabled (as module)
          Optional (for encrypted networks):
          - CONFIG_CRYPTO: enabled
          - CONFIG_CRYPTO_AEAD: enabled
          - CONFIG_CRYPTO_GCM: enabled (as module)
          - CONFIG_CRYPTO_SEQIV: enabled
          - CONFIG_CRYPTO_GHASH: enabled (as module)
          - CONFIG_XFRM: enabled
          - CONFIG_XFRM_USER: enabled
          - CONFIG_XFRM_ALGO: enabled
          - CONFIG_INET_ESP: enabled (as module)
          - CONFIG_INET_XFRM_MODE_TRANSPORT: enabled (as module)
      - "ipvlan":
        - CONFIG_IPVLAN: missing
      - "macvlan":
        - CONFIG_MACVLAN: enabled (as module)
        - CONFIG_DUMMY: enabled (as module)
      - "ftp,tftp client in container":
        - CONFIG_NF_NAT_FTP: enabled (as module)
        - CONFIG_NF_CONNTRACK_FTP: enabled (as module)
        - CONFIG_NF_NAT_TFTP: enabled (as module)
        - CONFIG_NF_CONNTRACK_TFTP: enabled (as module)
    - Storage Drivers:
      - "aufs":
        - CONFIG_AUFS_FS: missing
      - "btrfs":
        - CONFIG_BTRFS_FS: enabled (as module)
        - CONFIG_BTRFS_FS_POSIX_ACL: enabled
      - "devicemapper":
        - CONFIG_BLK_DEV_DM: enabled (as module)
        - CONFIG_DM_THIN_PROVISIONING: enabled (as module)
      - "overlay":
        - CONFIG_OVERLAY_FS: enabled (as module)
      - "zfs":
        - /dev/zfs: missing
        - zfs command: missing
        - zpool command: missing

    Limits:
    - /proc/sys/kernel/keys/root_maxkeys: 1000000
