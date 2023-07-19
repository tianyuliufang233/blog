---
title: "Kubernetes Service 负载均衡和联网"
date: 2023-08-09T16:38:13+08:00
categories:
- 项目
- kubernetes相关
tags:
- kubernetes
- 记录
keywords:
- service
#thumbnailImage: //example.com/image.jpg
---
service 负载均衡和联网
<!--more-->
# kubernetes网络模型
从端口分分配、命名、服务发现、负载均衡、应用配置和迁移的角度看，Pod可以被视为虚拟机或者物理主机。
- Pod要能与其他节点上的Pod通信，且不需要网络地址转译
- 节点上的代理(系统守护进程、kubelet)可以和节点上的所有Pod通信。
kubernetes 网络解决四方面问题：
- 一个Pod中的容器之间通过本地回路(loopback)通信
- 集群网络在不同Pod之间提供通信。
- service api 运行你向外暴露Pod中运行的应用，以支持来自于集群外部的访问。
  - ingress提供专门用于暴露HTTP应用程序、网站和API的额外功能。
- 可以使用Service来发布仅供集群内部使用的服务
## service
kubernetes中Service是将运行在一个或一组Pod上的网络应用程序公开为网络服务的方法。service定义的抽象能够解耦关联。service由Pod定义的选择符来确定。也会有不带选择符的service。ingress将路由规则整合到单个资源。

云原生服务发现是通过kubernetes api进行服务发现，可以查询API服务器用于匹配EndpointSlices。只要Pod发生变化，Kubernetes会更新服务EndpointSlices。

定义service：
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```
### EndpointSlices
EndpointSlices对象表示针对服务的后端网络端点的子集(切片)。当EndpointSlice表示的端点数量达到阈值时，kubernetes会添加另一个空的EndpointSlice并在其中存储新的端点信息。默认100个。
### Endpoints
该资源定义了网络端点的列表，通常由service引用，以定义可以将流量发送到哪些Pod。
kubernetes限制单个Endpoints对象中可以容纳的端点数量。当一个service配置超过1000个后备端点时，kubernetes会截断EndpointSlice。appProtocol字段提供了一种为每个Service端口指定应用协议的方式。
### 多端口Service
对于某些服务，需要开多个端口
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```
### 发布服务（服务类型）
可能希望将其暴露给kubernetes集群外部的IP地址。
- ClusterIP 只能内部访问，定义的.spec.clusterIP为None时，kubernetes不会分配IP地址。
- NodePort 通过每个节点上的IP和静态端口暴露服务。由.spec.ports[*].nodePort字段中报告已分配的端口。（默认：30000-32767）
- LoadBalancer 云提供的负载均衡器暴露服务。当type的值为"LoadBalancer"时，由云的负载均衡器直接重定向到后端以提供服务。
- ExternalName 将服务映射到exterName字段的内容。该映射将集群的DNS服务器配置为返回具有该外部主机名值的CNAME记录。无需创建任何类型代理。
#### 混合协议类型的负载均衡器
more对于LoadBalancer类型的服务，当定义了多个端口时，所有端口必须具有相同的协议，并且该协议必须是受云提供商支持的协议。多个端口时，需要开启MixedProtocolLBService.
#### 禁用负载均衡器节点端口分配
spec.allocateLoadBalancerNodePorts设置为false可以对LoadBalancer的服务禁用节点端口分配。这仅适用于直接路由到Pod的负载均衡器实现。默认为true。对于已配置的节点端口，设置为false不会自动释放。必须显式删除nodePorts项以释放对应端口。
#### 设置负载均衡器实现的类别
.spec.loadBalancerClass如果未设置，如果集群使用--cloud-provider配置云提供商，LoadBalancer类型会使用云提供商的默认负载均衡器实现。
#### 内部负载均衡器
```
腾讯云
metadata:
  annotations:
    service.kubernetes.io/qcloud-loadbalancer-internal-subnetid: subnet-xxxxx
```
#### ExternalName
将ExternalName类型的服务将service name 映射到DNS。
###  Headless Services
由spec.clusterIP的值为"None"来创建Headless Service。
#### 带选择算符的服务
对定义了选择算符的无头服务，kubernetes控制平面在kubernetes API中创建EndpointSlince对象，并修改DNS配置返回A或AAAA条记录，这些记录直接指向Service的后端Pod集合。
#### 无选择算符的服务
对于没有定义选择符的无头服务，控制平面不会创建EndpointSlice对象。DNS系统会查找以下其中一项配置：
- 对于type:ExternalName服务，查找合配置其CNAMW记录
- 对所有其他类型的服务，针对service的就绪端点的所有IP地址，查找合配置DNS A/AAAA记录
  - 对于IPv4端点，DNS系统创建A记录
  - 对于IPv6端点，DNS系统创建AAAA记录
定义无选择符的无头服务时，port必须与targetPort匹配。
### 服务发现
对于集群内运行的客户端，支持两种发现模式：环境变量和DNS。

一个service redis-primary暴露了TCP端口6379，同时被分配了Cluster IP地址，这个Service生成的环境变量：
```
REDIS_PRIMARY_SERVICE_HOST=10.0.0.11
REDIS_PRIMARY_SERVICE_PORT=6379
REDIS_PRIMARY_PORT=tcp://10.0.0.11:6379
REDIS_PRIMARY_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_PRIMARY_PORT_6379_TCP_PROTO=tcp
REDIS_PRIMARY_PORT_6379_TCP_PORT=6379
REDIS_PRIMARY_PORT_6379_TCP_ADDR=10.0.0.11
```
当Pod需要访问某service，并且使用环境变量方式将端口和集群IP发布的到客户端Pod时，必须在客户端Pod出现之前创建该service，否则，Pod中不会出现变量。
#### DNS
DNS服务器监视Kubernetes API中的新Service，并为每个Service创建一组DNS记录。DNS还支持命名端口的DNS SRV记录。DNS服务器是唯一能够访问ExternalName类型的Service的方式。
#### 虚拟IP和代理
kube-proxy组件负责除type为ExternalName以外的服务，实现虚拟IP机制。使用代理转发的几个原因：
- DNS的实现不遵守记录的TTL约定，在记录超期后可能任然有结果缓存。
- 有些应用只做一次DNS缓存，然后永久缓存结果。
- 即使应用程序和库进行了适当的重新解析，TTL值较低或为零的DNS记录可能会给DNS带来很大的压力，从而变得难以管理。
kube-proxy可以支持cleanup功能对规则进行清理。默认情况下iptables会随机选择一个后端。
#### 会话粘性
需要通过设置.spec.sessionAffinity为ClientIP来设置基于客户端的会话亲和性(默认为None)。会话粘性超时时间.spec.sessionAffinityConfig.clientIP.timeoutSeconds来设置大会话粘性时间（默认为10800，即3小时）。
#### 流量策略
.spec.internalTrafficPolicy字段控制来自内部源的流量如何被路由。有效值为Cluster和Local。将字段设置为Cluster会将内部流量路由到所有准备就绪的端点，将字段设置为Local只将流量路由到本地节点准备就绪的端点。如果流量策略为Local但没有本地节点端点，那么kube-proxy会丢掉该流量。.spec.externalTrafficPolicy控制外部流量。externalTrafficPolicy: Local的连接会先检查本地pod状态，如果没有或正在中止则转发到其他节点。
#### 外部IP
.spec.externalIPs指定外部IP地址。如果外部IP(作为目的IP地址)和端口都与service匹配，则kubernetes配置的规则和路由会确保流量被路由到该Service的端点。
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 49152
  externalIPs:
    - 198.51.100.32
```
## ingress
Ingress是对集群中服务的外部访问进行管理的API对象，典型的访问方式是HTTP。
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```
### 路径类型
- ImplementationSpecific 匹配方法取决于IngressClass。具体实现可以将其作为单独的pathType处理或与Prefix或Exact类型作相同处理。
- Exact  精确匹配URL路径，且区分大小写
- Prefix  基于/分隔的URl路径前缀匹配。匹配区分大小写，并且对路径中的元素逐个完成。对路径中的元素逐个匹配。如果路径的最后一个元素是请求路径中最后一个元素的子字符串，则不会匹配。
多重匹配情况下最长匹配路径优先。精确匹配优先于模糊匹配。主机名可以使用通配符匹配，但只匹配一个元素段。
#### IngressClass的作用域
取决于IngressClass控制器。IngressClass的参数默认是集群范围的。如果设置了.spec.parameters字段且未设置.spec.parameters.scope字段，或是.spec.parameters.scope字段设为了Cluster，那么该IngressClass所代指的就是一个集群作用域的资源。如果在.spec.parameters字段将.spec.parameters.scope字段设为Namespace,.spec.parameters.namespace则必须和此资源所处命名空间相同。
```
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb-1
spec:
  controller: example.com/ingress-controller
  parameters:
    # 此 IngressClass 的配置定义在一个名为 “external-config-1” 的
    # ClusterIngressParameter（API 组为 k8s.example.net）资源中。
    # 这项定义告诉 Kubernetes 去寻找一个集群作用域的参数资源。
    scope: Cluster
    apiGroup: k8s.example.net
    kind: ClusterIngressParameter
    name: external-config-1
---
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: external-lb-2
spec:
  controller: example.com/ingress-controller
  parameters:
    # 此 IngressClass 的配置定义在一个名为 “external-config” 的
    # IngressParameter（API 组为 k8s.example.com）资源中，
    # 该资源位于 “external-configuration” 命名空间中。
    scope: Namespace
    apiGroup: k8s.example.com
    kind: IngressParameter
    namespace: external-configuration
    name: external-config
```
## ingress control
维护Ingress资源工作。
## EndpointSlice
为Endpoints提供一种可扩缩和可拓展的替代方案。EndpointSlice包含对一组网络端点的引用。控制面自动为设置了选择符的Kubernetes Service创建EndpointSlice。这些EndpointSlice将包含对Service选择算符匹配的所有Pod的引用。EndpointSlice通过唯一的协议、端口号和Service名称将网络端点组织在一起。
```
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: example-abc
  labels:
    kubernetes.io/service-name: example
addressType: IPv4
ports:
  - name: http
    protocol: TCP
    port: 80
endpoints:
  - addresses:
      - "10.1.2.3"
    conditions:
      ready: true
    hostname: pod-1
    nodeName: node-1
    zone: us-west2-a
```
EndpointSlice支持三种地址类型：
- IPv4
- IPv6
- FQDN（完全合规域名）
每个EndpointSlice对象代表一个特定的IP地址类型。如果有一个支持IPv4和IPv6的Service，那么将至少有两个EndpointSlice对象（一个用于IPv4，一个用于IPv6）.
EndPointSlice 的端点状态有三种：ready、serving和terminating。
- ready就绪  当Pod处于运行中时，ready状况被设置为True。Pod处于终止过程中时，ready永远不为true。
- Serving 服务中  与read状况几乎相同，区别是它不考虑终止状态。
- Termination (终止中) 对于Pod来说，这是设置了删除时间戳的Pod
#### 拓扑信息
每个端点都可以包含一定的拓扑信息。如：nodeName,zone。
endpointslice.kubernetes.io/managed-by可以用来标明哪个实体在管理某个EndpointSlice。EndpointSlice都由某个Service所有，该端点切片正是为该服务跟踪记录其端点。这一署主关系可以设置kubernetes.io/service-name标签来标明。
在某些场合，应用会创建定制的Endpoints资源。控制面对Endpoints资源进行映射的例外情况：
- Endpoints资源上标签endpointslice.kubernetes.io/skip-mirror值为true。
- Endpoints资源包含标签control-plane.alpha.kubernetes.io/leader。
- 对应的Service资源不存在。
- 对应的Service选择符不存在。
每个子网最多有1000个地址被加入到EndpointSlice中。
#### EndpointSlices的分布问题
每个EndpointSlices都有一组端口值，适用于资源内的所有端点。当为Service使用命名端口时，Pod可能会就统一命名端口获得不同的端口号，因而需要不同的EndpointSlice。控制面尝试尽量将EndPointSlice填满，不过不会主动在若干EndpointSlice之间执行再平衡操作。与多个EndpointSlice更新操作相比，执行一个EndpointSince的创建操作的方案会更优先。EndpointSlice每次变更都变得代价更高。
由于EndpointSlice变化的自身特点，端点可能同时出现再不止一个EndPointSlice中。
## 网络策略
可以使用NetworkPolicy对3层或4层。NetworkPolicy是一种以应用为中心的结构。pod可以通信的标识：
- 其他被允许的Pods（Pod无法阻塞对自身的访问）
- 被允许的命名空间
- IP组块 （例外：与Pod运行所在的节点的通信一直被允许的，无论Pod或节点的IP地址）
网络策略通过网络插件实现。
### 隔离分为出口隔离和入口隔离
网络策略是相加的，不会产生冲突。
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```
选择器：namespaceSelector 选择器将特定的名字空间，podSelector 选择器将再与NetworkPolicy相同的名字空间中选择特定的Pod。ipBlock：将选择特定的ip范围作为入栈流量来源或出栈目的地。
#### SCTP支持
默认开启，要禁用需要在API server指定--feature-gates=SCTPSuppert=false禁用。必须使用支持SCTP的CNI插件。针对某个端口范围：
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: multi-port-egress
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 32000
          endPort: 32768
```
按标签选择多个命名空间：
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: egress-namespaces
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Egress
  egress:
   - to:
     - namespaceSelector:
       matchExpressions:
       - key: namespace
         operator: In
         values: ["frontend", "backend"]
```
#### 基于名字指向某命名空间
需要指定kubernetes.io/metadata.name。该标签的值是命名空间的名称。
## service与Pod的DNS
service的命名空间：podname.namespace.svc.cluster.local.

除了无头service之外的"普通"Service会被赋予一个如my-svc.my-namespace.svc.cluster-domain.example的DNS A和/或AAAA记录。没有集群IP的无头Service会被赋予形如my-svc.my-namespace.svc.cluster-domain.example 的 DNS A 和/或 AAAA 记录。

Pod A/AAAA记录 pod-ip-address.my-namespace.pod.cluster-domain.example。可用ip访问：172-17-0-3.default.pod.cluster.local。

Pod主机名取自Pod的metadata.name。可以在spec.hostname设置，spec.subdomain可以表明该Pod的命名空间的子组的一部分。类似于foo.bar.my-namespace.svc.cluster-domain.example。
```
apiVersion: v1
kind: Service
metadata:
  name: busybox-subdomain
spec:
  selector:
    name: busybox
  clusterIP: None
  ports:
  - name: foo # 实际上不需要指定端口号
    port: 1234
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox1
  labels:
    name: busybox
spec:
  hostname: busybox-1
  subdomain: busybox-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox2
  labels:
    name: busybox
spec:
  hostname: busybox-2
  subdomain: busybox-subdomain
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    name: busybox
```
Pod setHostnameAsFQDN字段为true时，kubelet会将Pod的全限域名（FQDN）作为该Pod的主机名记录到Pod的所在命名空间。这种情况下，hostname和hostname --fqdn都会返回busybox-1.busybox-subdomain.my-namespace.svc.cluster-domain.example。linux中，内核的主机名字段最多64字符。
#### Pod的DNS策略
- Defualt  从运行的所在节点继承名称解析配置。
- ClusterFirst  与配置的集群域后缀不匹配的任何DNS查询都会由DNS服务器转发到上游DNS服务器。可以配置额外的存根域和上游DNS服务器。
- ClusterFirstWithHostNet  对于以hostNetwork方式运行的Pod，应将其DNS策略显式设置ClusterFirstWithHostNet。否则以hostNetwrok方式和"ClusterFirst"策略运行的Pod将回退至"Default"策略的行为。
- None  此设置允许Pod忽略Kubernetes环境中的DNS设置。Pod会使用其dnsConfig字段所提供的DNS设置。
Default 不是默认DNS策略，未指定则使用ClusterFirst。
```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox:1.28
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
```
#### Pod的DNS配置
pod的DNS配置可以让用户对Pod的DNS设置进行更多控制。dnsConfig字段可选，它可以与任何dnsPolicy设置一起使用。但当Pod的dnsPolicy设置为None时，必须指定dnsConfig字段。
- nameservers  将用作DNS server的IP地址列表。最多可以指定3个IP地址。当Pod的dnsPolicy设置为None时，列表必须至少包含一个IP地址，否则此属性是可选的。所列出的服务器将合并到从指定的DNS策略生成的基本名称服务器，并删除重复的地址。
- searches: 用于Pod中查找主机名的DNS搜索域的列表。此属性是可选的。指定此属性时，所提供的列表将合并到根据所选DNS策略生成的基本搜索域名中。重复的域名将被删除。最多6个搜索域。
- options：可选对象列表，每个对象可能具有name属性（必需）和value属性（可选）。此属性的内容将合并到从指定的DNS策略生成的选项。重复的条目将被删除。
```
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 192.0.2.1 # 这是一个示例
    searches:
      - ns1.svc.cluster-domain.example
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```
kubernetes本身不限制DNS配置，最多可支持32个搜索域列表，所有搜索域的总长度不超过2048。此限制分别适用于节点的解析器配置文件、Pod的Dns配置和合并的DNS配置。
## IPv4/IPv6双协议栈
从1.21版本开始，默认启用IPv4/IPv6双协议栈，以支持同时分配IPv4和IPv6。
### 配置IPv4/IPv6双协议栈
- kube-apiserver: --service-cluster-ip-range=<IPv4>,<IPv6>
- kube-controller-manager:
  - --cluster-cidr=<IPv4 CIDR>,<IPv6 CIDR>
  - --service-cluster-ip-range=<IPv4 CIDR>,<IPv6 CIDR>
  - --node-cidr-mask-size-ipv4|--node-cidr-mask-size-ipv6 对于IPv4默认为/24，对于IPv6默认为/64
- kube-proxy: --cluster-cidr=<IPv4 CIDR>,</IPv6 CIDR>
- kubelet:
  - 当没有--cloud-provider时，管理员可以通过--node-ip来传递逗号分隔的IP地址，为该节点手动配置双栈.status.addresses。如果Pod以HostNetwork模式在该节点商运行，则Pod会用.status.podIPs字段来报告它的IP地址。一个节点的所有podIP都会匹配该节点的由.status.addresses字段定义的IP组。

使用外部云驱动时，如果kubelet和外部云提供商都启用了CloudDualStackNodeIPs特性门控，则可以将双栈--node-ip值传递给kubelet。此特性需要保证云提供商支持双栈集群。
### service IPv4或IPv6
服务器的地址默认为第一个服务集群IP范围地址族(通过kube-apiserver的--service-cluster-ip-range参数配置)。当定义服务时，可以配置为双栈。若要指定所需的行为，可以设置.spec.ipFamilyPolicy字段：
- SingleStack: 单栈服务。控制面使用第一个配置的服务集群IP范围为服务分配集群IP。
- PreferDualStack：为服务分配IPv4和IPv6集群地址。
- RequireDualStack: 从IPv4和IPv6的地址范围分配服务的.spec.ClusterIPs。从基于.spec.ipFailies数组中第一个元素的地址族.spec.ClusterIPs列表中选择.spec.ClusterIP
如果想要定义哪个IP族用于单栈或定义双栈IP族的顺序，可以通过设置服务上的可选字段.spec.ipFamilies来选择地址族，.spec.ipFamilies可以添加或删除第二个地址族，但不能更改现有服务的主IP地址族。
#### 新服务双栈选型
未显式设置.spec.ipFamilyPolicy。当创建服务时，kubernetes从配置的第一个service-cluster-ip-range中为服务分配一个集群IP，并设置.spec.ipFamilyPolicy为SingleStack。无选择符服务和无头服务的行为方式与此相同。
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app.kubernetes.io/name: MyApp
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
```
显式设置.spec.ipFamilyPolicy为PreferDualStack。当在双栈集群上创建此服务时，Kubernetes会为该服务分配IPv4和IPv6地址。.spec.ClusterIPs是主要字段，包含两个分配的IP地址，.spec.ClusterIP是次要字段，其取值从.spec.ClusterIPs计算而来。
- 对于.spec.ClusterIP字段，控制面记录来自第一个服务集群IP范围对应的地址族的IP地址。
- 对于单协议集群，.spec.ClusterIPs和.spec.ClusterIP字段都仅列出一个地址。
- 对于启用了双协议栈的集群，将.spec.ipFamilyPolicy设置为RequireDualStack时，其行为与PreferDualStack相同。
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app.kubernetes.io/name: MyApp
spec:
  ipFamilyPolicy: PreferDualStack
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
```
显式指定.spec.ipFamilies中指定IPv6和IPv4，并将.spec.ipFamilyPolicy设定为PreferDualStack。当Kubernetes为.spec.ClusterIPs分配一个IPv6和一个IPv4地址时，.spec.ClusterIP被设置成IPv6地址，因为它是.spec.ClusterIPs数组中的第一个元素，覆盖其默认值。
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app.kubernetes.io/name: MyApp
spec:
  ipFamilyPolicy: PreferDualStack
  ipFamilies:
  - IPv6
  - IPv4
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
```
## 拓扑感知路由
拓扑感知路由（Toplogy Aware Routing）调整路由行为，以优先保持流量在其发起区域内。在某些情况下，有助于降低成本或提高网络性能。
### 动机
kubernetes集群多地部署在多区域环境中。拓扑感知路由提供了一种机制帮助流量保留在其发起所在区域内。计算服务（service）的端点时，EndpointSlice控制器考虑每个端点的物理拓扑（地区和区域），并填充提示字段以将其分配到区域。诸如kube-proxy等集群组件可以使用这些提示，影响流量的路由方式（优先考虑物理拓扑上更近的点）。
通过service.kubernetes.io/topology-mode注解设置为Auto来启用Service的拓扑感知路由。当某个区域中有足够的端点可用时，系统将为EndpointSlices填充拓扑提示，把每个端点分配给特定区域，从而使流量被路由到更接近其来源的位置。
### 什么情况下效果最好
1. 入站流量均匀分布
当预计入栈流量源自同一区域时，不建议使用此特性。会导致端点子集过载。
2. 服务在每个区域具有至少三个端点
在一个三区域的集群中，意味着至少9个端点。如果每个区域的端点少于3个，则EndpointSlice控制器很大概率无法平均分配端点，回退到默认的集群范围的路由方法。
### 工作原理
"自动"启发算法会尝试按比例分配一定数量的端点到每个区域。这种启发方式对有大量端点的Service效果最佳。这种启发方式对具有大量端点的Service效果最佳。
### EndPointSlice控制器
当启用此启发方式时，EndpointSlice控制器负责在各个EndpointSlice上设置提示信息。控制器按比例给每个区域分配一定比例数量的端点。
```
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: example-hints
  labels:
    kubernetes.io/service-name: example-svc
addressType: IPv4
ports:
  - name: http
    protocol: TCP
    port: 80
endpoints:
  - addresses:
      - "10.1.2.3"
    conditions:
      ready: true
    hostname: pod-1
    zone: zone-a
    hints:
      forZones:
        - name: "zone-a"
```
### kube-proxy组件
kube-proxy组件依据EndpointSlice控制器设置的提示，过滤由它负责路由的端点。在大多数场合，kube-proxy可以把流量路由到同一个区域的端点。有时，控制器在另一不同的区域中分配端点，以确保在多个区域之间更平均地分配端点。这会导致部分流量被路由到其他区域。
### 保护措施
拓扑感知会有一些保护措施规则。如果规则无法顺利通过，kube-proxy将无视区域限制，从集群中的任意位置选择端点。
- 端点数量不足：如果一个集群中，端点数量少于区域数量，控制器不创建任何提示。
- 不可能实现均衡分配：在一些场合中，不可能实现端点在区域中的平衡分配。如：zone-a比zone-b大两倍，但只有2个端点，则分配到zone-a的端点可以能收到比zone-b多两倍的流量。如果控制器不能确保此"期望的过载"值低于每一个区域可接受的阈值，控制器将不添加提示信息。不是基于实时反馈，对于特定节点，可能超载。
- 一个或多个Node信息不足：如果任一节点没有设置标签topology.kubernetes.io/zone，或没有上报可分配的cpu数据，控制平面将不好设置任何拓扑感知提示，进而kube-proxy就不能根据区域来过滤端点。
- 至少一个端点没有设置区域提示：当这种情况发生时，kube-proxy会假设从拓扑感知提示到拓扑感知路由的迁移仍在进行中，在这种场合下过滤Service的端点是有风险的，kube-proxy回退到使用所有端点。
- 提示中不存在某区域：如果kube-proxy无法找到提示中指向它当前所在区域的端点，它将回退到使用来自所有区域的端点。当你向现有集群新增新的区域时，这种情况发送概率很高。
### 限制
- 当service的internalTrafficPolicy值设置为Local时，系统将不使用拓扑感知提示信息。可以在同一集群的不同service上使用这两个特性，但不能在统一service上这样做。
- 这种方法不适用于大部分流量来自于一部分区域的Service。这项技术的假设是入站流量于各区域中节点的服务能力成比例关系。
- EndpointSlice控制器忽略设置了node-role.kubernetes.io/control-plane或node-role.kubernetes.io/master标签的节点。如果负载也在这些节点上，也可能产生问题。
- EndpointSlice控制器在分派或计算各区域的比例时，并不会考虑容忍度。如果Service背后的各Pod被限制只能运行在集群节点的一个子集上，计算比例时不好考虑这点。
- 这项技术和自动扩缩容机制之间可能存在冲突。如：大量流量来源于同一个区域，那只有分配到该区域的端点才可用来处理流量。这导致Pod自动水平扩缩容要么不能正确处理这种场景，要么会在别的区域添加Pod。

## windows网络
## Service ClusterIP分配
在kubernetes中，Service是一种抽象方式，用于公开在一组Pod上运行的应用。有两种分配方式：
- 动态分配，由集群控制面自动从所配置的IP范围为type: ClusterIP选择一个空闲IP地址。
- 静态分配，根据Service所配置的Ip范围，选定并设置你的IP地址。整个集群中，每个service的ClusterIP都必须是唯一的。尝试使用已分配的ClusterIP创建Service将返回错误。
### 预留Service的ClusterIP
作为一种非强制性的约定，一些kubernetes安装程序将Service IP范围中的第10个IP地址分配给DNS服务。假设将集群的Service IP范围配置为10.96.0.0/16，并且希望Dns Service IP为10.96.0.10，则yaml如下:
```
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: CoreDNS
  name: kube-dns
  namespace: kube-system
spec:
  clusterIP: 10.96.0.10
  ports:
  - name: dns
    port: 53
    protocol: UDP
    targetPort: 53
  - name: dns-tcp
    port: 53
    protocol: TCP
    targetPort: 53
  selector:
    k8s-app: kube-dns
  type: ClusterIP
```
如果在DNS启动之前或同时采用动态分配机制创建其他Service，则它们有可能被分配此IP，因此，你将无法创建DNS Service,因为它会因冲突错误而失败。默认情况下，动态IP分配使用地址较高的一段，一旦用完，将使用较低范围。这将允许用户在冲突风险较低地段上使用静态分配。
## 服务内部流量策略
为Service的.spec.internalTrafficPolicy项设置为Local时，Service的yaml:
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  internalTrafficPolicy: Local
```
kube-proxy基于spec.internalTrafficPolicy的设置来过滤路由的目标服务端点。当设置为Local时，只会选择节点本地端点。当值设为Cluster或缺省时，Kubernetes会选择所有的服务端点。

