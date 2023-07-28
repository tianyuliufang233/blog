---
title: "Kubernetes 卷"
date: 2023-07-27T11:58:09+08:00
categories:
- category
- subcategory
tags:
- tag1
- tag2
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

## cephfs
允许将现有的fs挂载到pod中。在pod被删除时被保留，只是被卷卸载。
## configMap
提供了向pod注入配置数据的方法。configMap对象可以被挂载为卷。
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox:1.28
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```
## downloadAPI
在这类卷中，所公开的数据以纯文本格式的只读文件形式存在。
说明： 容器以 subPath 卷挂载方式使用 downward API 时，在字段值更改时将不能接收到它的更新。
## emptyDir
emptyDir会随pod创建和删除。容器崩溃不好导致pod被从节点上移除。
emptyDir的一些用途：
  - 缓存空间
  - 耗时较长的计算任务提供检查点，以便任务能方便地从崩溃前状态恢复执行。
  - 在web服务器容器服务数据时，保存内容管理器容器获取的文件。 

emptyDir.medium字段用来控制emptyDir卷的存储位置。
## fc(光纤通道)
允许将光纤通道块挂载到pod中。可以设置参数targetWWNs指定单个或多个目标WWN(World Wide Names)

## hostPath
hostPath 存在安全风险，尽量避免使用HostPath。
hostPath的一些用法：
  - 允许需要访问docker内部机制的容器
  - 允许容器中运行cAdvisor
  - 允许Pod指定给定的hostPath在运行Pod之前是否应该存在。
可以指定的type如下：
|值|行为|
|---|---|
||空字符串（默认）用于向后兼容，这意味着在安装 hostPath 卷之前不会执行任何检查。|
|DirectoryOrCreate|如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 kubelet 相同的组和属主信息。|
|Directory|在给定路径上必须存在的目录。|
|FileOrCreate|如果在给定路径上什么都不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 kubelet 相同的组和所有权。|
|File|在给定路径上必须存在的文件|
|Socket|在给定路径上必须存在的UNIX套接字|
|CharDevice|在给定路径上必须存在的字符设备|
|BlockDevice|在给定路径上必须存在的块设备|

host FileOrcreate模式不会负责创建文件的父目录。如果挂载的父级目录不存在，pod启动会失败。
```
apiVersion: v1
kind: Pod
metadata:
  name: test-webserver
spec:
  containers:
  - name: test-webserver
    image: registry.k8s.io/test-webserver:latest
    volumeMounts:
    - mountPath: /var/local/aaa
      name: mydir
    - mountPath: /var/local/aaa/1.txt
      name: myfile
  volumes:
  - name: mydir
    hostPath:
      # 确保文件所在目录成功创建。
      path: /var/local/aaa
      type: DirectoryOrCreate
  - name: myfile
    hostPath:
      path: /var/local/aaa/1.txt
      type: FileOrCreate
```

## iscsi
能将iscsi卷挂载到pod中。只能单个写模式挂载，能够多个使用者读模式挂载。
## local
代表某个被挂载的本地存储设备，例如：磁盘、分区或者目录。
只能用作静态创建的持久卷。不支持动态配置。
与hostPath相比，能以持久和可移植的方式使用，而无需手动将Pod调度到节点。通过查看PV的节点亲和性配置，就能了解卷的节点约束。
设置local卷时，需要设置PV的nodeAffinity属性。使用storageClass时建议设置volumeBindingMode为WaitForFirstConsumer。
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```

## nfs
nfs卷能将nfs网络系统挂载到pod中。
```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /my-nfs-data
      name: test-volume
  volumes:
  - name: test-volume
    nfs:
      server: my-nfs-server.example.com
      path: /my-nfs-volume
      readOnly: true
```
## persistentVolumeClaim
将持久卷PersistentVolume挂载到Pod中。
## Portworx CSI 迁移
针对Portworx添加CSIMigration特性，在1.23中默认禁用，1.25中已进入Beta阶段，默认关闭。它将所有插件操作不在指向树内插件(In-Tree Plugin),转而指向容器存储接口(Container Storage Interface，CSI)驱动。要启用此特性，需要在kube-controller-manager 和kubelet中设置 CSIMigrationPortworx=true 。
## rbd
允许将Rados块设备卷挂载到Pod中。rbd在删除Pod时被保存，卷只是被卸载。
只能单写多读形式安装。
## RBD CSI 迁移
启用RBD的CSIMigration特性后，所有插件操作从现有的树内插件重定向到CSI驱动程序。要使用该特性需要启用xsiMigrationRBD
## secret
用于给Pod传递敏感信息。以文件形式挂载到Pod中，由tmpfs(基于RAM的文件系统)提供存储。
## vSphere CSI 迁移
在1.27中，对树内vsphereVolume类的所有操作都被重定向到csi.vsphere.vmware.com CSI驱动程序。
vSphere CSI驱动不支持：
-  diskformat
-  hostfailurestotolerate
-  forceprovisioning
-  cachereservation
-  diskstripes
-  objectspacereservation
-  iopslimit
## subPath
在单个Pod中共享卷以提供多方使用时很有用。volumeMounts.subPath属性用于指定所引用的卷内的子路径，而不是根路径。
```
apiVersion: v1
kind: Pod
metadata:
  name: my-lamp-site
spec:
    containers:
    - name: mysql
      image: mysql
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "rootpasswd"
      volumeMounts:
      - mountPath: /var/lib/mysql
        name: site-data
        subPath: mysql
    - name: php
      image: php:7.0-apache
      volumeMounts:
      - mountPath: /var/www/html
        name: site-data
        subPath: html
    volumes:
    - name: site-data
      persistentVolumeClaim:
        claimName: my-lamp-site-data
```

### 扩展环境变量的subPath
subPathExpr可以基于download API环境变量来构造subPath目录名。subPath和subPathExpr属性是互斥的。
```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container1
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    image: busybox:1.28
    command: [ "sh", "-c", "while [ true ]; do echo 'Hello'; sleep 10; done | tee -a /logs/hello.txt" ]
    volumeMounts:
    - name: workdir1
      mountPath: /logs
      # 包裹变量名的是小括号，而不是大括号
      subPathExpr: $(POD_NAME)
  restartPolicy: Never
  volumes:
  - name: workdir1
    hostPath:
      path: /var/log/pods
```
## 资源
emptyDir的存储介质由保存kubelet数据的根目录(/var/lib/kubelet)的文件系统的介质确定。
## out-of-tree 卷插件
Out-of-tree卷插件包括容器存储接口(CSI)。使得存储供应商能够创建自定义存储插件。无需将插件源码添加kubernetes仓库。
## CSI
容器存储接口为容器编排系统定义标准接口，以将任意系统暴露给它们的容器工作负载。
CSI的三种使用方式：
- pvc对象引用
- 一般性临时存卷
- CSI临时卷，前提是驱动可以支持这种用法
CSI持久卷配置字段：
- driver：指定卷驱动名称的字符串值
- volumeHandle: 唯一标识卷的字符串值
- readOnly: 一个可选的布尔值
- fsType: 如果PV的VolumeMode为Filesystem，那么此字段指定挂载时应该使用的文件系统。
- volumeAttributes: 一个字符串到字符串的映射表，用来设置卷的静态属性。
- controllerPublishSecetRef: 对包敏感信息的secret对象的引用；该敏感信息会被传递给CSI驱动来完成CSI ControllerPublishVolume和ControllerUnpublishVolume调用。
- nodeExpandSecretRef: 对包含敏感信息的Secret对象的引用。
- nodePublishSecretRef: 对包含敏感信息的Secret对象的引用，该信息会传递CSI驱动以完成CSI NodeStageVolume调用。
- nodeStageSecretRef: 对敏感信息的Secret对象的引用，该信息会传递给CSI驱动以完成CSI NodeStageVolume调用。

## 挂载卷的传播
挂载传播特性由Continer.volumeMounts中的mountPropagation字段控制。
- None  不会感知到主机后续在此卷或其任何子目录上执行的挂载变化
- HostToContainer 此卷挂载将会感知到主机后续针对卷或其任何子目录的挂载操作。
- Bidirectional 这种挂载和HostToContainer 挂载表现相同。另外，容器创建的卷挂载将被传播回至主机和使用同一卷的所有Pod的所有容器。
Bidirectional可以破坏主机操作系统，因此只被允许特权容器中使用。

参考连接[kubernetes-volumes]("https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#configmap")