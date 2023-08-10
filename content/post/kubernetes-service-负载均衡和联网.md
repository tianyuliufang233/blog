---
title: "Kubernetes Service 负载均衡和联网"
date: 2023-08-09T16:38:13+08:00
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

## IPv4/IPv6双协议栈
## 拓扑感知路由
## windows网络
## Service ClusterIP分配
## 服务内部流量策略
