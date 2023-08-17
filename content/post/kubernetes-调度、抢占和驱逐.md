---
title: "Kubernetes 调度、抢占和驱逐"
date: 2023-08-15T11:32:46+08:00
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
调度、抢占和驱逐
<!--more-->
## 调度
### kubernetes调度器
通过Kubernetes的监测(Watch)机制来发现集群中新创建且尚未被调度到节点上的Pod。调度器会将所发现的每一个为调度的Pod调度到一个合适的节点上允许。调度器会依据调度原则来做出调度选择。
如果没有任何一个节点能满足Pod的资源请求，那么这个Pod将一直停留在未调度状态直到调度器能够找到合适的Node。

调度器先在集群中找到一个Pod的所有可调度节点，然后根据一系列函数对可调到节点打分，选出其中得分最高的节点来运行Pod。之后，调度器将这个调度决定通知给kube-apiserver，这个国产叫做绑定。
#### kube-scheduler 中的节点选择
kube-scheduler给一个Pod做调度选择时，包含两个步骤：过滤、打分。
过滤阶段会将所有满足Pod调度需求的节点选出来。如：PodFitsResources过滤函数会检查候选节点的可用资源能否满足Pod的资源请求。过滤后，得出一个节点列表，里面包含所有可调度节点。打分阶段，调度器会为Pod从所有可调度节点根据当前的打分规则进行打分，选取出最合适的节点。

支持两种方式配置调度器的过滤和打分行为：
- 调度策略 允许配置过滤锁用的断言(Predicates)和打分所用的优先级(Priorities)
- 调度配置 允许配置实现不同调度阶段的插件，包括：QueueSort、Filter、Score、Bind、Reserve、Permit等等。
### 将Pod指派到节点
可以约束一个Pod以便限制其只能在特定的节点上运行，或优先在特定的节点上运行。
#### 节点隔离/限制
通过为节点加标签，可以准备让Pod调度到特定节点或节点组上。可以使用这个功能来确保特定的Pod只能运行在具有一定隔离性、安全性或监管属性的节点上。
如果使用标签来实现节点隔离，建议选择节点上的kubelet无法修改的标签键。这可以防止受感染的节点在自身上设置这些标签，进而影响调度器将工作负载调度到受感染的节点。NodeRestricton准入插件防止kubelet使用node-restriction.kubernetes.io/前缀设置或修改标签。
要使用该标签前缀进行节点隔离：
- 确保在使用节点鉴权机制并且已经启用了NodeRestriction准入插件。
- 将带有node-restriction.kubernetes.io/前缀的标签添加到Node对象，然后在节点选择符中使用这些标签。
#### nodeSelector
是节点选择约束最简单的方式。
#### 亲和性和反亲和性
亲和性和反亲和性扩展了约束类型。一些好处：
- 亲和性、反亲和性的表达能力更强。
- 可以标明某规则是"软需求"或"偏好"，这样调度器在无法找到匹配节点时仍然调度该Pod。
- 可以使用节点上运行的其他Pod的标签来实施调度约束，
亲和性功能由两种类型的亲和性组成：节点亲和性类似于nodeSelector字段，但它的表达能力更强，并且允许指定软规则。Pod键亲和性/反亲和性允许根据其他Pod的标签来约束Pod。
#### 节点亲和性
- requiredDuringSchedulingIgnoredDuringExecution: 调度器只有在规则被满足的时候才能执行调度。此功能类似于nodeSelector。
- preferredDuringSchedulingIgnoredDuringExecuting：调度器会尝试寻找满足对应规则的节点。如果找不到匹配的节点，调度器仍然会调度该Pod。
IgnoredDuringExecution意味着如果节点标签在kubernetes调度Pod后发生了变更，Pod仍将继续运行。可以使用Pod规约中的.spec。affinity.nodeAffinity字段设置节点亲和性。
```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: topology.kubernetes.io/zone
            operator: In
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
```
如果同时指定了nodeSelector和nodeAffinity，两者必须都要满足，才能将Pod调度到候选节点上。如果在nodeAffinity类型关联的nodeSelectorTerms中指定多个条件，只要其中一个nodeSelectorTerms满足（各个条件逻辑或操作组合）的话，Pod就可以被调度到节点上。如果nodeSelectTerms中的条件相关的单个matchExpressions字段中指定多个表达式，则只有当所有表达式都满足是，Pod才能被调度到节点上。
#### 节点亲和性权重
可以为preferredDuringSchedulingIgnoredDuringExecution亲和性类型的每个实例设置weight字段，其取值范围是1到100。
```
apiVersion: v1
kind: Pod
metadata:
  name: with-affinity-anti-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: label-1
            operator: In
            values:
            - key-1
      - weight: 50
        preference:
          matchExpressions:
          - key: label-2
            operator: In
            values:
            - key-2
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
```
必须拥有打了kubernetes.io/os=linux标签的节点。
#### 逐个调度方案中设置节点亲和性
在配置多个调度方案时，可以将某个方案与节点亲和性相关联起来。可以在调度器配置中为NodeAffinity插件的args字段添加addEdAffinity。如：
```
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
  - schedulerName: foo-scheduler
    pluginConfig:
      - name: NodeAffinity
        args:
          addedAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: scheduler-profile
                  operator: In
                  values:
                  - foo
```
DaemonSet控制器为DaemonSet创建Pod，但该控制器不理会调度方案。DaemonSet控制器创建Pod时，默认的Kubernetes调度器负责防止Pod，并遵从DaemonSet控制器中设置的nodeAffinity规则。

#### Pod间亲和性与反亲和性
Pod间亲和性与反亲和性使你可以基于已经在节点上运行的Pod的标签来约束Pod可以调度到的节点，而不是基于节点上的标签。Pod间亲和性与反亲和性都需要相当的计算量，因此在大规模集群中显著降低调度速度。不建议超过百个节点的集群中使用这类设置。

Pod反亲和性需要节点上存在一致性的标签。换言之，集群中每个节点都必须拥有与topologykey匹配的标签。如果某些或者所有节点上不存在指定的topologyKey标签，调度行为可能与预期的不同。

Pod间亲和性与反亲和性的类型：
- requireDuringSchedulingIgnoreedDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution
对于亲和性可以使用.affinity.podAffinity字段，对于反亲和性使用.affinity.podAntiAffinity字段。
案例：
```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: registry.k8s.io/pause:2.0
```
原则上topologyKey可以是任何合法的标签键。出于性能和安全原因，topologyKey有一些限制：
- 对于Pod亲和性而言，在requiredDuringSchedulingIgnoredDuringExecution和preferredDuringSchedulingIgnoredDuringExecution中，topologyKey不允许为空。
- 对于requiredDuringSchedulingIgnoredDuringExecution要求的Pod反亲和性，准入控制器LimitPodHardAntiAffinityToplogy要求topologyKey只能是kubernetes.io/hostname。如果使用其他拓扑逻辑，可以更改准入控制器或禁用。

除了labelSelector和topologyKey,可以在labelSelector和topologyKey的同一层上设置namespace以指定labelSelector要匹配的命名空间列表。如果namespace被忽略或为空，则默认为Pod亲和性/反亲和性的定义所在的命名空间。
#### 命名空间选择符
用户可以使用namespaceSelector选择匹配的命名空间，此参数是对命名空间集和进行标签查询的机制。空的{}会匹配所有空间，null或namespace为null则是当前Pod的命名空间。

### Pod开销
### Pod拓扑分布约束
### 污点和容忍度
### 动态资源分配
### 调度框架
### 调度器性能调试
### 扩展资源的资源装箱
### Pod调度就绪
### Descheduler

## Pod 干扰
### Pod优先级和抢占
### 节点压力驱逐
### API发起的驱逐

