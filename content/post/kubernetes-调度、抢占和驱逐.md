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
- 如果新的Pod的toplogySpreadConstraints[*].labelSelector与自身的标签不匹配。更新工作负载的 topologySpreadConstraints[*].labelSelector 以匹配 Pod 模板中的标签。
#### 集群级别的默认约束
为集群设置默认的拓扑分布约束也是可能的。默认拓扑分布约束在且仅在以下条件下才会被应用到Pod上：
- Pod没有在其.spec.topologySpreadConstraints中定义任何约束。
- Pod隶属于某个Service、ReplicaSet、StatefulSet或ReplicatonController。
默认约束可以设置为调度方案中PodToplogySpread插件参数的一部分。labelSelector必须为空。
```
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
    pluginConfig:
      - name: PodTopologySpread
        args:
          defaultConstraints:
            - maxSkew: 1
              topologyKey: topology.kubernetes.io/zone
              whenUnsatisfiable: ScheduleAnyway
          defaultingType: List
```
默认SelectorSpread插件是被禁用的。Kubernetes项目建议使用PodTopologySpread以执行类似行为。
#### 内置默认约束
```
defaultConstraints:
  - maxSkew: 3
    topologyKey: "kubernetes.io/hostname"
    whenUnsatisfiable: ScheduleAnyway
  - maxSkew: 5
    topologyKey: "topology.kubernetes.io/zone"
    whenUnsatisfiable: ScheduleAnyway
```
用于提供同等行为的SelectorSpread插件默认被禁用。如果节点不会同时设置kubernetes.io/hostname和topology.kubernetes.io/zone标签，应该定义自己的约束而不是使用Kubernetes的默认约束。如果不想为集群使用默认的Pod分布约束，可以通过设置defaultingType参数为List，并将PodTopologySpread插件配置中的defaultConstraints参数置空来禁用Pod分布约束：
```
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration

profiles:
  - schedulerName: default-scheduler
    pluginConfig:
      - name: PodTopologySpread
        args:
          defaultConstraints: []
          defaultingType: List
```
#### 比较podAffinity和podAntiAffinity
在Kubernetes中，Pod间亲和性和反亲和性控制Pod彼此的调度方式(更密集或更分散)。
- podAffinity：吸引Pod；可以尝试将任意数量的Pod集中到符合条件的拓扑域中。
- podAntiAffinity：驱逐Pod。如果设置为requiredDuringSchedulingIgnoredDuringExecution模式，则只有单个Pod可以调度到单个拓扑域；如果选择preferredDuringScheduleIgnoredDuringExecution,则将丢失强制执行此约束的能力。
#### 局限性
- 当Pod被移除时，无法保证约束仍然被满足。当Deployment的规模缩减时，Pod的分布可能不再均衡。可以使用Descheduler来重新实现Pod分布的均衡。
- 具有污点的节点上匹配的Pod也会被统计。
- 调度器不会预先知道集群拥有的所有可用区和其他拓扑域。拓扑域由集群中存在的节点确定。在自动扩缩容的集群种，如果一个节点池的节点数量缩减为零，而用户期望其扩容时，可能导致调度出现问题。因为这种情况下，调度器不会考虑这些拓扑域，直至这些拓扑域中至少包含一个节点。
### 污点和容忍度
节点亲和性被吸引到一类特定的节点。污点（Taint）则相反。
容忍度（Toleration）是应用于Pod上的。容忍度允许调度器调度带有对应污点的Pod。容忍度允许调度但并不保证调度：作为其功能的一部分，调度器会评估其他参数。
```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
```
operator的默认值是Equal。
一个容忍度和一个污点相"匹配"是它们有一样的键名和效果，并且：
- 如果operator是Exists（此时容忍度不能指定value）
- 如果operator是Equal，则它们的value应该相等。
特殊情况：
- 如果一个容忍度的key为空且operator为Exists，表示这个容忍度与任意的key、value和effect都匹配，即这个容忍度能容忍任何污点。
- 如果effect为空，则可以与所有键名的效果相匹配。
需要注意的情况：
- 如果未被忽略的污点中存在至少一个effect值为NoSchedule的污点，则Kubernetes不会将Pod调度到该节点。
- 如果未被忽略的污点中不存在effect值为NoSchedule的污点，但是存在至少一个effect值为PreferNoSchedule的污点，则Kubernetes会尝试不将Pod调度到该节点。
- 如果未被忽略的污点中存在至少一个effect值为NoExecute的污点，则kubernetes不会将Pod调度到该节点。(如果Pod还未在节点上允许)，并且会将Pod从该节点驱逐(如果Pod已经在节点上运行)
一些内置污点：
- node.kubernetes.io/not-ready：节点未准备好。这相当于节点状况Ready的值为"False"。
- node.kubernetes.io/unreachable：节点控制器访问不到节点，相当于节点Ready的值为Unknown
- node.kubernetes.io/memory-pressure：节点存在内存压力
- node.kubernetes.io/disk-pressure：节点存在磁盘压力
- node.kubernetes.io/pid-pressure：节点的PID压力
- node.kubernetes.io/network-unavailable：节点网络不可用
- node.kubernetes.io/unschedulable：节点不可调度
- node.cloudprovider.kubernetes.io/uninitialized：如果kubelet启动时指定外部平台驱动，它将给当前节点添加一个污点将其标志为不可用。
节点被排空时，当节点不可达时，API服务器无法与节点上的kubelet进行通信。在与API服务器的通信被重新简历之前，删除Pod的决定无法传递到kubelet。同时，被调度进行删除的Pod可能继续运行在分区后的节点上。
当节点不可达时，API服务器无法与节点上的kubelet进行通信。在与API服务器的通信被重新建立之前，删除Pod的决定无法传递到kubelet。同时，被调度进行删除的Pod可能会继续运行在分区后的节点上。
控制面会限制向节点添加新污点的速度。这一速率限制可以管理多个节点同时不可达时，可能触发的驱逐的数量。

Pod可设置tolerationSecods，以指定当节点失败或者不响应时，Pod维系与该节点间绑定关系的时长。当出现网络分裂事件时，对于一个与节点本地状态有深度绑定的应用，仍然停留在当前节点上运行一段较长时间，等待网络恢复以避免被驱逐。这种容忍度的配置案例如下：
```
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```
DaemonSet中的Pod被创建时，针对以下污点会自动添加NoExecute的容忍度不会指定tolerationSeconds：
- node.kubernetes.io/unreachable
- node.kubernetes.io/not-ready
以保证DaemonSet的Pod永远不会被驱逐。
#### 基于节点状态添加污点
DaemonSet控制器自动为所有收获进程添加NoSchedule容忍度，防止DaemonSet崩溃：
- node.kubernetes.io/memory-pressure
- node.kubernetes.io/disk-pressure
- node.kubernetes.io/pid-pressure
- node.kubernetes.io/unschedulable
- node.kubernetes.io/network-unavailable（只适合主机网络配置）
### 调度框架
调度框架是面向kubernetes调度器的一种插件架构。
框架定义一些扩展点。注册后在一个或多个扩展点被调用。这些插件中的一些可以改变调度策略，另一些用于提供信息。每次调度一个Pod的尝试都分为两个阶段，调度周期和绑定周期。
#### 扩展点
##### PreEnqueue
这些插件在将Pod添加到内部互动队列之前被调用，在此对队列中Pod被标记为准备好进行调度。只有所有PreEnqueue返回Success时，Pod才允许进入活动队列。否则，被放置到内部无法调度的Pod列表中，并且不会获得Unschedulable状态。
##### 队列排序
这些插件用于对调度队列中的Pod进行排序。队列排序插件本质上提供Less(Pod1，Pod2)函数。一次只能启动一个队列插件。
##### PreFilter
预处理Pod的相关信息，或者检查集群或Pod必须满足的某些条件。如果PreFilter插件返回错误，则调度周期将终止。
##### Filter
这些插件用于过滤出不能运行该Pod的节点。对于每个节点，调度器按照顺序调用过滤插件。如果任何过滤插件将节点标记为不可行，则不会为该节点调用剩下的过滤插件。节点可以被同时进行评估。
##### PostFilter
这些插件在Filter阶段后调用，但仅在该Pod没有可行的节点时调用。插件按其配置的顺序调用。如果任何PostFilter插件标记节点为"Schedulable"，则其余的插件不会调用。PostFilter实现是抢占，试图通过抢占其他Pod的资源使该Pod可以调度。
##### PreScore
插件用于执行"前置评分（pre-scoring）"工作，即生成一个可共享状态供Score插件使用。如果PreScore插件返回错误，则调度周期终止。
##### Score
通过过滤阶段的节点进行排序。调度器将为每个节点调用每个评分插件。将有一个定义明确的整数范围，代表最小和最大分数。在标准化评分阶段后，调度器将根据配置的插件权重合并所有插件的节点分数。
##### NormalizeScore
这些插件用于在调度器计算Node排名之前修改分数。在此扩展点注册的插件被调用时会使用同一插件的Score结果。每个插件在每个调度周期调用一次。
##### Reserve
Reserver和Unreserv，他们分别支持两个名为Reserve和Unreserve的信息处理性质的调度阶段。维护运行时状态的插件("有状态插件")应该使用这两个阶段，以便在节点上的资源被保留和未保留给特定的Pod时得到调度器的通知。Reserve阶段发生在调度器实际将一个Pod绑定到其指定节点前。它的存在是为了防止在调度器等待绑定成功时发生竞争情况。每个Reserve插件的Reserve方法调用失败，后面的插件就不会被执行。操作会被认为是失败的。如果Reserve阶段或后续阶段失败了，则触发了Unreserve阶段。所有Reserve插件的Unreserve方法将按照Reserve方法调用的相反顺序执行。这个阶段为了清理与保留的Pod相关的状态。Unreserve方法的实现必须是幂等的，并且不能失败。
##### Permit
Permit插件在每个Pod调度周期的最后调用，用于防止或延迟Pod的绑定。一个允许插件可以做的事:
- 批准：一旦所有Permit插件批准Pod后，该Pod将被发送以进行绑定
- 拒绝：如果任何Permit插件拒绝Pod，该Pod将被返回到调度队列。将触发Reserve插件的Unreserve阶段。
- 等待：如果一个Permit插件返回"等待"结果，则Pod将保持在一个内部的"等待中"的Pod列表，同时该Pod的绑定周期启动时即直接阻塞直到批准。如果超时发生，等待变成拒绝，并且Pod将返回调度队列，从而触发Reserve插件中的Unreserve阶段。
##### PreBind
用于执行Pod绑定前所需的所有工作。如制备网络卷并且挂载到目标节点。
##### Bind
Bind插件用于将Pod绑定到节点上。直到所有的PreBind插件都完成，Bind插件才会被调用。按照顺序被调用。Bind插件可以选择是否处理指定的Pod。如果某Bind插件选择处理某Pod，剩余的Bind插件将被跳过。
##### PostBind
在Pod成功绑定后被调用。
##### Unreserve
这是个信息性的扩展点。如果Pod被保留，在后面的阶段中被拒绝，则Unreserve插件将被通知。Unreserve插件应该清楚保留Pod的相关状态。
### 动态资源分配
是一个用于在Pod之间和Pod内部容器之间请求和共享资源的API。对为通用资源所提供的持久卷API的泛化。第三方资源驱动程序负责跟踪和分配资源。不同类型的资源支持用任意参数进行定义和初始化。
### 调度器性能调试
kube-scheduler主要负责将Pod调度到集群的Node上。可以通过调整percentageOfNodesScore来配置节点的阈值。percentageOfNodesToScore选项接收从0到100之间的整数值。0表示编译后的默认值。超过100等价于100。
#### 节点打分阈值
可以使用整个集群节点总数的百分比作为阈值来指定需要多少节点。kube-scheduler会将它转换为节点数的整数值。在调度期间，如果kube-scheduler已确认的可调度节点数足以超过配置的百分比数量，kube-scheduler将停止继续查找可调度节点并继续进行打分阶段。
#### 默认阈值
100-节点下取50%，5000-节点集群下取10%。这个自动设置的参数最低值是5%。
#### percentageOfNodesToScore参数
值必须在1-100之间，默认值是通过集群的规模计算得来。当集群少于50个时，调度器任然去检查所有Node，因为可调度节点太少，不足以停止调度器最初的过滤选择。如果将percentageOfNodesToScore设置一个较低值，则没有效果或效果很小。
该参数设置后，可能导致只有集群种少数节点被选为可调度节点，很多节点都没有进入到打分阶段。由于这个原因，这个参数不应该设置为一个很低的值。通常做法是不会将这个参数的值设置得低于10。太低的参数值一般在调度器的吞吐量很高且对节点的打分不重要的情况下才使用。
#### 调度器做调度选择的时候如何覆盖所有Node
为了让集群种所有节点都有公平的机会去运行Pod，调度器将会以轮询的方式覆盖全部的Node。可以理解为Node列表想象为数组。调度器从数组的头部开始筛选可调度节点，直到percentageOfNodesToScore参数的要求数量。
### 扩展资源的资源装箱（bin packing）
MostAllocated和RequestedToCapacityRatio。两种资源策略。
#### 使用MostAllocated策略启用资源装箱
MostAllocated策略基于资源的利用率来为节点计分，优先分配比率较高的节点。
使用插件NodeResourcesFit设置MostAllocated策略的案例：
```
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration
profiles:
- pluginConfig:
  - args:
      scoringStrategy:
        resources:
        - name: cpu
          weight: 1
        - name: memory
          weight: 1
        - name: intel.com/foo
          weight: 3
        - name: intel.com/bar
          weight: 3
        type: MostAllocated
    name: NodeResourcesFit
```
#### 使用RequestedToCapacityRatio策略来启用资源装箱
RequestedToCapacityRatio策略允许用户基于请求值与容量的比率，针对参与节点计分的每类资源设置权重。这些策略使得用户可以使用合适的参数来对扩展资源执行装箱操作，进而提升大规模集群中稀有资源的的利用率。
NodeResourcesFit计分函数中的RequestedToCapacityRatio可以通过字段scoringStrategy来控制。可以配置requestedToCapacityRatio和resources。requestedToCapacityRatio参数中的shape设置使得用户能够调整函数的算法，基于utilization和score值计算最少请求或最多请求。resources参数中包含计分过程中需要考虑的资源的name，以及用来设置每种资源权重的weight。

```
apiVersion: kubescheduler.config.k8s.io/v1beta3
kind: KubeSchedulerConfiguration
profiles:
- pluginConfig:
  - args:
      scoringStrategy:
        resources:
        - name: intel.com/foo
          weight: 3
        - name: intel.com/bar
          weight: 3
        requestedToCapacityRatio:
          shape:
          - utilization: 0
            score: 0
          - utilization: 100
            score: 10
        type: RequestedToCapacityRatio
    name: NodeResourcesFit
```
## Pod 干扰
### Pod优先级和抢占
Pod可以有优先级。优先级表示一个Pod相对其他Pod的重要性。
### 节点压力驱逐
### API发起的驱逐

