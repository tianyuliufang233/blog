---
title: "Kubernetes 卷"
date: 2023-07-27T11:58:09+08:00
categories:
- 收件箱
- kubernetes相关
tags:
- kubernetes
- 记录
keywords:
- 卷
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
# 持久卷
持久卷（PersistentVolume，pv）是集群中的一块存储，可以由管理员事先制备，或者存储类（StorageClass）来动态制备。PV持久卷和普通Volume一样，使用卷插件来实现。
持久卷申领（PersistentVolumeClaim，PVC）表达的是用户对存储的请求。概念上与Pod类似。Pod消耗节点资源，PVC消耗PV资源。
## 制备
PV分为静态和动态制备
### 静态制备
集群管理员创建若干PV卷。这些卷带有真实存储的细节信息，对集群用户可见。
### 动态制备
基于StorageClass来实现的：PVC申领请求某个存储类，必须已经创建并配置该类才能动态制备PV。需要在API服务器上启用DefaultStorageClass准入控制器。可通过API server组件的--enable-admission-plugins标志值实现这点。
## 绑定
用户创建一个带有特定存储容量和特定访问模式需要的PVC对象；在动态制备场景下，制备的PVC和PV是一对一映射。如果找不到匹配的PV卷，PVC会无限期处于未绑定状态。当与之匹配的PV卷可用时，PVC申领会被绑定。
### 保护使用中的存储对象
当PVC为Terminating且Finalizers列表中包含kubernetes.io/pvc-protetion时，PVC处于被保护状态。
### 回收（Reclaiming）
当用户不再使用其存储卷时，可以从API中将PVC对象删除，从而允许该资源被回收再利用。PV的回收策略告诉集群，当其被从申领中释放时如何处理该数据卷。目前的几种状态Retained（保留）、Recycled（回收）或Deleted（删除）。
### 保留（Retain）
回收策略Retain使得用户可以手动回收资源。当PVC被删除时，PV卷任然存在，对应的数据卷被视为"以释放（released）"。需要手动回收该卷：
- 删除PV。
- 根据情况手动清除关联存储资产上的数据。
- 手动删除关联的存储资产。
### 删除（Delete）
对于支持Delete回收策略的卷插件，删除动作将PV删除，同时移除关联的存储资产。动态制备的卷会继承StorageClass中设置的回收策略。默认为Delete。
### PV删除保护finalizer
可以在PV上添加Finalizer，以确保只有在删除对应的存储后才删除具有Delete回收策略的PV。
kubernetes.io/pv-controller和external-provisioner.volume.Kubernetes.io/finalizer可被添加到动态制备的卷上。
### 预留PV
通过PVC中指定PV，可以声明特定PV与PVC之间的绑定关系。如果PV存在未被通过其claimRef字段预留给PVC，则该PV会和该PVC绑定到一起。绑定操作不会考虑某些卷匹配条件是否满足，包括节点亲和性等。任然会检查存储类、访问模式和请求的存储大小是否合法。

使用claimPolicy属性设置为Retain的PersistentVolume卷时，包括希望复用现有PV时，很有用。

## 扩充PVC申领
支持扩充类型的卷：
- azureDisk
- azureFile
- awsElasticBlockStore
- csi
- flexVolume
- gcePersistentDisk
- rbd
- portworxVolume
PVC的存储类中将allowVolumeExpansion设置为true时，才可以扩充该PVC。设置一个更大的尺寸值。这一操作会触发下层PV提供存储的卷的扩充。kubernetes不会创建新的PV来满足请求。现有的卷会被调整大小。
直接边际PV的大小可以阻止该卷自动调整大小。如果对PV的容量进行编辑，然后又对PVC的.spec进行编辑，使得PVC大小匹配PV的话，则不会发送存储大小调整。控制面观察到资源匹配，并认为其后备卷大小已手动增加，无需调整。
### CSI卷扩充
CSI扩充能力默认启用，扩充CSI卷要求CSI驱动支持卷扩充操作。
#### 重设包含文件系统的卷的大小
只有卷中包含文件系统是XFS、EXT3或EXT4时，才可以重设卷大小。当卷中包含文件系统时，只有Pod使用ReadWrite模式来使用PVC申领的情况下才能重设其文件系统的大小。文件系统扩充的操作可能是在Pod启动期间完成，或在下层文件系统支持在想扩充的前提下在Pod运行期间完成。
#### 重设使用中PVC的大小
所有使用中的PVC在其文件系统被扩充后，立即可供Pod使用。此特性对于没有被Pod或Deployment使用的PVC是无效的。必须在执行扩展操作之前创建一个使用该PVC的Pod。
#### 处理扩展卷过程失败
如果扩充下层存储失败，可以手动恢复PVC申领的状态并取消重设大小请求。否则会反复重试
- 将绑定到PVC的PV标记为Retain回收策略
- 删除PVC对象。由于PV的回收策略为Retain，我们不会重建PVC时丢失数据。
- 删除PV规约中的claimRef，这样新的PVC可以绑定到该卷。这一操作会使得PV卷变为"可用（Available）"。
- 使用小于PV卷大小的尺寸重建PVC，设置PVC的volumeName字段为PV卷的名称。这一操作将把新的PVC对象绑定到现有PV卷
- 恢复PV卷上设置的回收策略。
通过请求扩展为更小尺寸。此特性为alpha特性。RecoverVolumeExpansionFailure必须被启用以允许使用此特性。
当开启此特性后可通过扩展更小尺寸重试扩展。可以通过编辑.spec.resources选择一个更小值。可通过查看.status.resizeStatus以及PVC上的事件来监控调整大小操作的状态。不支持将pvc缩小到小于当前的尺寸。
### 持久卷类型
- cephfs
- csi
- fc
- hostPath
- iscsi
- local
- nfs
- rbd
### 持久卷
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```
卷模式支持两种：Filesystem和Block。默认Filesystem。
访问模式：ReadWriteOnce、ReadOnlyMany、ReadWriteMany、ReadWriteOncePod
卷访问模式并不能在存储已经被挂载的情况下实施写保护。
### 类
每个PV可以属于某个类，通过storageClassName属性设置为某个名称来指定。特定类的PV卷只能绑定到请求该类存储卷的PVC申领。
### 回收策略
- Retain 手动回收
- Recycle  基本擦除 rm -rf /thevolume/*
- Delete   删除关联资产。目前仅NFS和HostPath支持回收（Recycle）
### 挂载类型选型
- awsElasticBlockStore
- azureDisk
- azureFile
- cephfs
- gcePersistentDisk
- iscsi
- nfs
- rbd
- vsphereVolume
### 节点亲和性
可以通过设置节点亲和性来定义一些约束，限制从哪些节点上可以访问此卷。使用这些卷的Pod只会被调度到节点亲和性规则所选择的节点上执行。要设置节点亲和性，设置PV卷.spec中的nodeAffinity。
### 阶段
每个卷会处于以下阶段(Phase)之一：
- Available（可用）卷是一个空闲资源，尚未绑定到任何申领
- Bound（已绑定）该卷已经绑定到某申领
- Released（已释放）所绑定的申领已被删除，但资源尚未被集群回收
- Failed（失败）卷的自动回收操作失败
### 选择符
- matchLabels  卷必须包含带有此值的标签
- matchExpressions  通过设定键、值列表和操作符来构造的需求。合法的操作符有In、Notin、Exists和DoesNotExist。
## 类
通过storageclass.kubernetes.io/is-default-class赋值为true来完成。
基于block的卷
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: block-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  persistentVolumeReclaimPolicy: Retain
  fc:
    targetWWNs: ["50060e801049cfd1"]
    lun: 0
    readOnly: false
```
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: block-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-block-volume
spec:
  containers:
    - name: fc-container
      image: fedora:26
      command: ["/bin/sh", "-c"]
      args: [ "tail -f /dev/null" ]
      volumeDevices:
        - name: data
          devicePath: /dev/xvda
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: block-pvc
```
## 卷快照与卷克隆
卷快照（Volume Snapshot）与卷克隆都只支持树外CSI卷插件。
# 投射卷
一个projected可以将若干现有的卷映射到同一个目录上。
可被投射的资源：
- secret
- downwardAPI
- configMap
- serviceAccountToken
所有的卷源都要求处于Pod所在的同一个命名空间。
```
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  containers:
  - name: container-test
    image: busybox:1.28
    volumeMounts:
    - name: all-in-one
      mountPath: "/projected-volume"
      readOnly: true
  volumes:
  - name: all-in-one
    projected:
      sources:
      - secret:
          name: mysecret
          items:
            - key: username
              path: my-group/my-username
              mode: 755
      - downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "cpu_limit"
              resourceFieldRef:
                containerName: container-test
                resource: limits.cpu
      - configMap:
          name: myconfigmap
          items:
            - key: config
              path: my-group/my-config
      - serviceAccountToken:
          audience: api
          expirationSeconds: 3600
          path: token
```
expirationSeconds字段是服务账号令牌预期的生命期长度。默认1小时，至少10分钟。也可以通过--service-account-max-token-expiration设置最大值上限。

Linux中SecurityContext设置RunAsUser属性的Pod中，投射文件具有正确的署主关系。kubelet将确保serviceAccountToken卷的内容归该用户所有，并且令牌文件的权限模式会被设置为0600。
# 临时卷
临时卷会遵从Pod的生命周期，与Pod一起创建和删除。Pod规约中，以内联方式定义，这简化了应用程序的部署和管理。
## 临时卷类型
支持的临时卷类型：
- emptyDir：Pod启动时为空，存储空间来自kubelet根目录或内存，由kubelet管理。
- configMap、downloadAPI、secret：将不同类型的Kubernetes数据注入到Pod中。由kubelet管理。
- CSI临时卷：类似于前面的卷类型，但由专门支持此类型的指定CSI驱动程序提供。
- 通用临时卷：它可以由所有支持持久卷的驱动程序提供
CSI临时卷必须由第三方CSI存储驱动程序提供。通用临时卷可以由第三方CSI存储驱动程序提供，也可以由支持动态制备的任何其他存储驱动程序提供。

CSI临时卷类似于configMap、downwardAPI和secret类型的卷：在各个本地节点管理卷的存储，并在Pod调度到节点后于其他本地资源一起创建。Pod的存储资源的限制只能由kubelet对其自己管理的存储强制执行。

CSI临时存储的Pod的清单：
```
kind: Pod
apiVersion: v1
metadata:
  name: my-csi-app
spec:
  containers:
    - name: my-frontend
      image: busybox:1.28
      volumeMounts:
      - mountPath: "/data"
        name: my-csi-inline-vol
      command: [ "sleep", "1000000" ]
  volumes:
    - name: my-csi-inline-vol
      csi:
        driver: inline.storage.kubernetes.io
        volumeAttributes:
          foo: bar
```
volumeAttributes决定驱动程序准备什么样的卷。
### 通用临时卷
类似于emptyDir卷，但也提供一些额外的特性：
- 存储可以是本地的，也可以是网络连接的
- 卷可以有固定的大小，Pod不能超量使用。
- 卷可能有一些初始数据，这取决于驱动程序和参数
- 支持典型的卷操作，前提是相关的驱动程序也支持该操作，包括快照、克隆、调整大小、存储容量跟踪。
```
kind: Pod
apiVersion: v1
metadata:
  name: my-app
spec:
  containers:
    - name: my-frontend
      image: busybox:1.28
      volumeMounts:
      - mountPath: "/scratch"
        name: scratch-volume
      command: [ "sleep", "1000000" ]
  volumes:
    - name: scratch-volume
      ephemeral:
        volumeClaimTemplate:
          metadata:
            labels:
              type: my-frontend-volume
          spec:
            accessModes: [ "ReadWriteOnce" ]
            storageClassName: "scratch-storage-class"
            resources:
              requests:
                storage: 1Gi
```
自动创建的PVC采取确定性的命名机制：Pod名称和卷名称的组合，中间由连字符-连接。
## 存储类
StorageClass包含provisioner、parameters和reclaimPolicy字段，这些字段会在StorageClass需要动态制备PersistentVolume时使用到。
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations: 
    storageclass.kubernetes.io/is-default-class: false
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```
### 回收策略
reclaimPolicy 字段中指定回收策略，默认是Delete，可以是Retain。
allowVolumeExpansion为可扩展 设置为true则可扩展。
mountOptions指定挂载选项，如果卷不支持挂载选项，指定后会失败。
### 绑定模式
volumeBindingMode字段控制了卷绑定和动态制备应该发生在什么时候。默认为Immediate模式。
Immediate表示一旦创建了卷PV也就完成了绑定和动态制备。对于由于拓扑限制而非集群所有节点可达的存储后端，PV会在不知道Pod调度要求的情况下邦迪或制备。
可以通过WaitForFirstConsumer模式解决这个问题。
```
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  nodeSelector:
    kubernetes.io/hostname: kube-01
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```
可以使用allowedTopologies设置制备拓扑结构。
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - us-central-1a
    - us-central-1b
```
## Ceph RBD
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/rbd
parameters:
  monitors: 10.16.153.105:6789
  adminId: kube
  adminSecretName: ceph-secret
  adminSecretNamespace: kube-system
  pool: kube
  userId: kube
  userSecretName: ceph-secret-user
  userSecretNamespace: default
  fsType: ext4
  imageFormat: "2"
  imageFeatures: "layering"
```
- monitors: Ceph monitor，逗号分隔。该参数是必须的。
- adminId: Ceph 客户端ID，用于在池ceph池中创建映像。默认admin
- adminSecret: adminId的Secret名称。该参数为必须。提供的secret必须有值为"kubernetes.io/rbd"的type参数。
- adminSecretNamespace:  adminSecret的命名空间。默认是default。
- pool：CephRBD池。默认是"rbd"
- userId: Ceph 客户端ID，用于映射RBD镜像。默认与adminId相同
- userSecretName: 用于映射RBD镜像的userId的Ceph Secret的名字。必须与PVC存在于相同的namespace中。该参数是必须的。提供的secret必须具有值为"kubernetes.io/rbd"的type参数。
- userSecretNamespace: userSecretName的命名空间。
- fsType: Kubernetes 支持的fsType。默认为"ext4"
- imageFormat: Ceph RBD 镜像格式，“1”或者“2”.默认为1
- imageFeatures：可选参数，只能在imageFormat设置为2时才使用。目前功能只有layering。默认未打开。
## 动态卷
使用动态卷
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 30Gi
```
设置默认StorageClass为默认，添加注解**ingressclass.kubernetes.io/is-default-class: "true"**
### 卷快照、卷快照class、CSI卷克隆
都基本都是要CSI，目前没有使用，就不记录了。
### 存储容量
从1.27开始对存储容量跟踪的集群级API支持。
#### API
两个API扩展接口：
- CSIStorageCapacity对象：由CSI驱动程序在安装驱动程序的命名空间中产生。每个对象包含一个存储类的容量信息，并定义哪些节点可以访问该存储。
- CSIDriverSpec.StorageCapacity字段：设置为true时，Kubernetes调度程序将考虑使用CSI驱动程序的卷的存储容量。
#### 调度
存储容量信息将会被kubernetes调度程序使用：
- Pod使用的卷还没有被创建
- 卷使用引用了CSI驱动的StorageClass，并且使用了WaitForFirstConsumer卷绑定模式
- 驱动程序的CSIDriver对象的StorageCapacity被设置为true
将卷的大小与CSIStorageCapacity对象中列出的容量进行比较，并使用包含该节点的拓扑。
对于有Immediate卷绑定模式的卷，存储驱动程序将决定在何处创建该卷，而不取决于将使用该卷的Pod。对于CSI临时卷，调度总是不在考虑存储容量的情况下进行。基于：该卷类型仅由节点本地的特殊CSI驱动程序使用，并且不需要大量资源。的考虑
单节点有卷数量限制。
#### 卷健康监测
卷健康检测是CSI驱动从底层的存储系统着手，探测异常的卷状态，以事件的形式上报到PVC或Pod。
如果要启动节点失效监测功能，可以设置enable-node-watcher为true。
