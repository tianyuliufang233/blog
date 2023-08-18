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
用户可以使用namespaceSelector选择匹配的命名空间，此参数是对命名空间集和进行标签查询的机制。空的{}会匹配所有空间，null或namespace为null则是当前Pod的命名空间。podAntiAffinity规则告诉调度器避免将多个带有标签的副本部署到同一个节点上。
#### nodeName
nodeName优先级高于nodeSelector或亲和性、非亲和性规则。
#### Pod拓扑分布约束
可以通过拓扑分布约束来控制Pod在集群中故障域之间的分布，故障域的示例有区域（Region）、可用区（zone）、节点和其他用户自定义的拓扑域。这样做有助于提升性能、实现高可用或提升资源利用率。
#### 操作符
|操作符|行为|
|---|---|
|In|标签值存在于提供的字符串集中|
|NotIn|标签值不包含在提供的字符串集中|
|Exists|对象上存在具有此键的标签|
|DoesNotExist|对象上不存在此键的标签|
以下操作符只能与nodeAffinity一起使用。不能与非整数一起使用。如果给定的值未解析为整数，则Pod将无法被调度。Gt和Lt不适用于podAffinity。
|操作符|行为|
|---|---|
|Gt|提供的值将被解析为整数，并且该整数小于通过解析此选择算符命名的标签的值所得到的整数|
|Lt|提供的值将被解析为整数，并且该整数大于通过解析此选择算符命名的标签的值所得到的整数|
### Pod开销
运行Pod时，Pod本身占用的大量系统资源。定义overhead字段的RuntimeClass开销。
```
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: kata-fc
handler: kata-fc
overhead:
  podFixed:
    memory: "120Mi"
    cpu: "250m"
```
### Pod调度就绪状态
Pod一旦创建就被认为准备好进行调度。通过指定或删除Pod的.spec.schedulingGates,可以控制Pod何时准备好被纳入考量进行调度。scheduler_pending_pods带有一个新标签"gated"，以区分Pod是否已尝试调度但被宣称不可调度，或明确标记为未准备好调度。
#### 可变Pod调度指令
在高层次上，只能收紧Pod调度指令。更新后端指令将导致Pod只能被调度到它之前匹配的节点子集上。具体如下：
- 对于.spec.nodeSelector，只允许增加。如果原来未设置，则允许设置此字段。
- 对于.spec.affinity.nodeAffinity，如果当前值未nil，则允许设置未任意值。
- 如果NodeSelectorTerms之前为空，则允许设置该字段。如果之前不为空，则仅允许增加NodeSelectorRequirements到matchExpressions或fieldExpressions，且不允许更改当前的matchExpressions和fieldExpressions。因为.requiredDuringSchedulingIgnoredDuringExecution.NodeSelectorTerms中的条目被执行逻辑或运算，而nodeSelectorTerms[].matchExpressions和nodeSelectorTerms[].fieldExpressions中的表达式被执行逻辑与运算。
- 对于.preferredDuringSchedulingIgnoredDuringExecution，所有更新都被允许。这是因为首选条目不具有权威性，因此策略控制器不会验证这些条目。
### Pod拓扑分布约束
可以使用拓扑分布约束控制Pod在集群内故障域之间的分布，例如区域（Region）、可用区（Zone）、节点和其他用户自定义拓扑域。可以为集群设置默认约束，或为个别工作负载配置拓扑分布约束。
topologySpreadConstraints字段
```
---
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  # 配置一个拓扑分布约束
  topologySpreadConstraints:
    - maxSkew: <integer>
      minDomains: <integer> # 可选；自从 v1.25 开始成为 Beta
      topologyKey: <string>
      whenUnsatisfiable: <string>
      labelSelector: <object>
      matchLabelKeys: <list> # 可选；自从 v1.27 开始成为 Beta
      nodeAffinityPolicy: [Honor|Ignore] # 可选；自从 v1.26 开始成为 Beta
      nodeTaintsPolicy: [Honor|Ignore] # 可选；自从 v1.26 开始成为 Beta
  ### 其他 Pod 字段置于此处
```
#### 分布约束定义
- maxSkew描述Pod可能被不均匀分布的程度。必须指定此字段且该数值大于零。其语义将随着whenUnsatisfiable的值发生改变：
  - 如果选择whenUnsatisfiable: DoNotSchedule，则maxSkew定义目标拓扑中匹配Pod的数量与全局最小值（符合条件的域中匹配的最小Pod数量，如果符合条件的域数量小于MinDomains则为零）之间的最大允许差值。
  - 如果选择whenUnsatisfiable: ScheduleAnyway，则该调度器会更偏西能够降低偏差值的拓扑域。
- minDomains表示符合条件的域的最小数量。此字段是可选的。域是拓扑的一个特定实例。符合条件的域是其节点与节点选择器匹配的域。此字段在1.25中默认禁用。
  - 指定的minDomains值必须大于0。可以结合whenUnsatisfiable: DoNotSchedule 仅指定minDomains。
  - 当符合条件的、拓扑键匹配的域的数量小于minDomains时，拓扑分布将"全局最小值"（global minimum）设为0，然后进行skew计算。"全局最小值"是一个符合条件的域中匹配Pod的最小数量，如果符合条件的域的数量小于minDomains，则全局最小值为零。
  - 当符合条件的拓扑键匹配域的个数等于或大于minDomains时，该值对调度没有影响。
  - 如果未指定minDomains，则约束行为类似于minDomains等于1。
- topologyKey是节点标签的键。标记相同的标签值则将这些节点视为处于同一拓扑域中。在拓扑域中的每个实例称为一个域。调度器将尝试在每个拓扑域中放置数量均衡的Pod。另外，符合条件的域定义未其节点满足nodeAffinityPolicy和nodeTaintsPolicy要求的域。
- whenUnsatisfiable指示如果Pod不满足分布约束时如何处理：
  - DoNotSchedule （默认）告诉调度器不要调度
  - ScheduleAnyway告诉调度器仍然继续调度，只是根据如何能将偏差最小化来对节点进行排序。
- labelSelector 用于查找匹配的Pod。匹配此标签的Pod将被统计，以确定相应拓扑域中Pod的数量。
- matchLabelKeys 是一个Pod标签键的列表，用于选择需要计算分布式的Pod集合。这些键与labelSelector进行逻辑与运算，以选择一组已有的Pod，通过这些Pod计算新来Pod的分布方式。matchLabelKeys和labelSelector中禁止存在相同的键。未设置labelSelector时无法设置matchLabelKeys。Pod标签中不存在的键将被忽略。null或空意味着仅与labelSelector匹配。
```
    topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: foo
          matchLabelKeys:
            - pod-template-hash
```
- nodeAffinityPolicy 表示在计算Pod拓扑分布偏差时将如何处理Pod的nodeAffinity/nodeSelector。 如果值为nil，此行为等同于Honor策略。选项为
  - Honor: 只有nodeAffinity/nodeSelector 匹配的节点才会包括到计算中。
  - Ignore: nodeAffinity/nodeSelector被忽略。所有节点均包括到计算中。
- nodeTaintsPolicy 表示在计算Pod拓扑分布偏差时将如何处理节点污点。值为nill则等同于Ignore策略。选项为：
  - Honor：包括不带污点的节点及污点被新Pod所容忍的节点。
  - Ignore：节点污点被忽略。包括所有节点。
#### 隐式约定
- 只有新来的Pod具有相同命名空间的Pod才能作为匹配候选者。
- 调度器会忽略没有任何topologySpreadConstraints[*].topologyKey的节点。意味着：
  - 位于这些节点上的Pod不影响maxSkew计算。
  - 新的Pod没有机会被调度到这类节点上。
- 如果新的Pod的toplogySpreadConstraints[*].labelSelector与自身的标签不匹配。
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

