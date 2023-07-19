---
title: "Kubernetes集群部署文档"
date: 2025-06-05T13:55:03+08:00
categories:
- 收件箱
- 记录
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->


# docker镜像
```
docker tag swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/cni:v3.28.2 172.27.0.7:9000/dockerio/calico/cni:v3.28.2
docker tag swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/node:v3.28.2 172.27.0.7:9000/dockerio/calico/node:v3.28.2
docker tag swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/calico/kube-controllers:v3.28.2 172.27.0.7:9000/dockerio/calico/kube-controllers:v3.28.2
docker tag swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/coredns/coredns:1.9.4 172.27.0.7:9000/dockerio/coredns/coredns:1.9.4
```


# kubernetes 各配置文件：
## etcd.conf.yml
```
# This is the configuration file for the etcd server.
 
# Human-readable name for this member.
name: 'etcd1'
 
# Path to the data directory.
data-dir: /data/data/etcd/default.etcd
 
# Path to the dedicated wal directory.
wal-dir:
 
# Number of committed transactions to trigger a snapshot to disk.
snapshot-count: 10000
 
# Time (in milliseconds) of a heartbeat interval.
heartbeat-interval: 100
 
# Time (in milliseconds) for an election to timeout.
election-timeout: 1000
 
# Raise alarms when backend size exceeds the given quota. 0 means use the
# default quota.
quota-backend-bytes: 0
 
# List of comma separated URLs to listen on for peer traffic.
listen-peer-urls: https://172.16.1.94:2380
 
# List of comma separated URLs to listen on for client traffic.
listen-client-urls: https://172.16.1.94:2379
 
# Maximum number of snapshot files to retain (0 is unlimited).
max-snapshots: 5
 
# Maximum number of wal files to retain (0 is unlimited).
max-wals: 5
 
# Comma-separated white list of origins for CORS (cross-origin resource sharing).
cors:
 
# List of this member's peer URLs to advertise to the rest of the cluster.
# The URLs needed to be a comma-separated list.
initial-advertise-peer-urls: https://172.16.1.94:2380
 
# List of this member's client URLs to advertise to the public.
# The URLs needed to be a comma-separated list.
advertise-client-urls: https://172.16.1.94:2379
 
# Discovery URL used to bootstrap the cluster.
discovery:
 
# Valid values include 'exit', 'proxy'
discovery-fallback: 'proxy'
 
# HTTP proxy to use for traffic to discovery service.
discovery-proxy:
 
# DNS domain used to bootstrap initial cluster.
discovery-srv:
 
# Initial cluster configuration for bootstrapping.
initial-cluster:
 
# Initial cluster token for the etcd cluster during bootstrap.
initial-cluster-token: 'etcd-cluster'
 
# Initial cluster state ('new' or 'existing').
initial-cluster-state: 'new'
 
# Reject reconfiguration requests that would cause quorum loss.
strict-reconfig-check: false
 
# Enable runtime profiling data via HTTP server
enable-pprof: true
 
# Valid values include 'on', 'readonly', 'off'
proxy: 'off'
 
# Time (in milliseconds) an endpoint will be held in a failed state.
proxy-failure-wait: 5000
 
# Time (in milliseconds) of the endpoints refresh interval.
proxy-refresh-interval: 30000
 
# Time (in milliseconds) for a dial to timeout.
proxy-dial-timeout: 1000
 
# Time (in milliseconds) for a write to timeout.
proxy-write-timeout: 5000
 
# Time (in milliseconds) for a read to timeout.
proxy-read-timeout: 0
 
client-transport-security:
  # Path to the client server TLS cert file.
  cert-file: /data/apps/kubernetes/ssl/etcd.pem
 
  # Path to the client server TLS key file.
  key-file: /data/apps/kubernetes/ssl/etcd-key.pem
 
  # Enable client cert authentication.
  client-cert-auth: true
 
  # Path to the client server TLS trusted CA cert file.
  trusted-ca-file: /data/apps/kubernetes/ssl/ca.pem
 
  # Client TLS using generated certificates
  auto-tls: false
 
peer-transport-security:
  # Path to the peer server TLS cert file.
  cert-file: /data/apps/kubernetes/ssl/etcd.pem
 
  # Path to the peer server TLS key file.
  key-file: /data/apps/kubernetes/ssl/etcd-key.pem
 
  # Enable peer client cert authentication.
  client-cert-auth: true
 
  # Path to the peer server TLS trusted CA cert file.
  trusted-ca-file: /data/apps/kubernetes/ssl/ca.pem
 
  # Peer TLS using generated certificates.
  auto-tls: false
 
# The validity period of the self-signed certificate, the unit is year.
self-signed-cert-validity: 1
 
# Enable debug-level logging for etcd.
log-level: info
 
logger: zap
 
# Specify 'stdout' or 'stderr' to skip journald logging even when running under systemd.
log-outputs: [stderr]
 
# Force to create a new one member cluster.
force-new-cluster: false
 
auto-compaction-mode: periodic
auto-compaction-retention: "1"
```
## kube-apiserver.conf
```
KUBE_APISERVER_OPTS="--enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \
 --anonymous-auth=false \
 --bind-address=172.16.1.94 \
 --secure-port=6443 \
 --advertise-address=172.16.1.94 \
 --insecure-port=0 \
 --authorization-mode=Node,RBAC \
 --runtime-config=api/all=true \
 --enable-bootstrap-token-auth \
 --service-cluster-ip-range=10.174.0.0/16 \
 --token-auth-file=/data/apps/kubernetes/conf/token.csv \
 --service-node-port-range=30000-50000 \
 --tls-cert-file=/data/apps/kubernetes/ssl/kube-apiserver.pem  \
 --tls-private-key-file=/data/apps/kubernetes/ssl/kube-apiserver-key.pem \
 --client-ca-file=/data/apps/kubernetes/ssl/ca.pem \
 --kubelet-client-certificate=/data/apps/kubernetes/ssl/kube-apiserver.pem \
 --kubelet-client-key=/data/apps/kubernetes/ssl/kube-apiserver-key.pem \
 --service-account-key-file=/data/apps/kubernetes/ssl/ca-key.pem \
 --service-account-signing-key-file=/data/apps/kubernetes/ssl/ca-key.pem  \
 --service-account-issuer=api \
 --etcd-cafile=/data/apps/kubernetes/ssl/ca.pem \
 --etcd-certfile=/data/apps/kubernetes/ssl/etcd.pem \
 --etcd-keyfile=/data/apps/kubernetes/ssl/etcd-key.pem \
 --etcd-servers=https://172.16.1.94:2379 \
 --enable-swagger-ui=true \
 --allow-privileged=true \
 --apiserver-count=3 \
 --audit-log-maxage=30 \
 --audit-log-maxbackup=3 \
 --audit-log-maxsize=100 \
 --audit-log-path=/data/log/kube-apiserver-audit.log \
 --event-ttl=1h \
 --alsologtostderr=true \
 --logtostderr=false \
 --log-dir=/data/log/kubernetes \
 --feature-gates=RemoveSelfLink=false \
 --v=4"
```
## kube-controller-manager.conf
```
KUBE_CONTROLLER_MANAGER_OPTS="--port=0 \
  --secure-port=10257 \
  --bind-address=0.0.0.0 \
  --kubeconfig=/data/apps/kubernetes/conf/kube-controller-manager.kubeconfig \
  --service-cluster-ip-range=10.174.0.0/16 \
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/data/apps/kubernetes/ssl/ca.pem \
  --cluster-signing-key-file=/data/apps/kubernetes/ssl/ca-key.pem \
  --allocate-node-cidrs=true \
  --cluster-cidr=10.175.0.0/16 \
  --experimental-cluster-signing-duration=87600h \
  --root-ca-file=/data/apps/kubernetes/ssl/ca.pem \
  --service-account-private-key-file=/data/apps/kubernetes/ssl/ca-key.pem \
  --leader-elect=true \
  --feature-gates=RotateKubeletServerCertificate=true \
  --controllers=*,bootstrapsigner,tokencleaner \
  --horizontal-pod-autoscaler-sync-period=10s \
  --tls-cert-file=/data/apps/kubernetes/ssl/kube-controller-manager.pem \
  --tls-private-key-file=/data/apps/kubernetes/ssl/kube-controller-manager-key.pem \
  --use-service-account-credentials=true \
  --alsologtostderr=true \
  --logtostderr=false \
  --log-dir=/data/log/kubernetes \
  --v=2"
```
## kubelet.json
```
{
  "kind": "KubeletConfiguration",
  "apiVersion": "kubelet.config.k8s.io/v1beta1",
  "authentication": {
    "x509": {
      "clientCAFile": "/data/apps/kubernetes/ssl/ca.pem"
    },
    "webhook": {
      "enabled": true,
      "cacheTTL": "2m0s"
    },
    "anonymous": {
      "enabled": false
    }
  },
  "authorization": {
    "mode": "Webhook",
    "webhook": {
      "cacheAuthorizedTTL": "5m0s",
      "cacheUnauthorizedTTL": "30s"
    }
  },
  "address": "0.0.0.0",
  "port": 10250,
  "readOnlyPort": 10255,
  "cgroupDriver": "systemd",                   
  "hairpinMode": "promiscuous-bridge",
  "systemReserved": {"cpu": "200m","memory": "4096Mi"},
  "serializeImagePulls": false,
  "clusterDomain": "cluster.local.",
  "clusterDNS": ["10.174.0.2"]
}
```
## kubelet.yml
```
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: "0.0.0.0"
port: 10250
serializeImagePulls: false
evictionHard:
  memory.available:  "200Mi"
  nodefs.available:  "10%"
  nodefs.inodesFree: "5%"
  imagefs.available: "15%"
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /data/apps/kubernetes/ssl/ca.pem
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
readOnlyPort: 10255
clusterDNS:
  - "10.174.0.2"
clusterDomain: "cluster.local."
cgroupDriver: "systemd"
systemReserved:
  cpu: "200m"
  memory: "1024Mi"
rotateCertificates: true
containerLogMaxSize: 1Gi
containerLogMaxFiles: 5
```
kube-proxy.yaml
```
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
clientConnection:
  kubeconfig: /data/apps/kubernetes/conf/kube-proxy.kubeconfig
clusterCIDR: 10.175.0.0/16
healthzBindAddress: 0.0.0.0:10256
kind: KubeProxyConfiguration
metricsBindAddress: 0.0.0.0:10249
mode: "ipvs"
```
kube-scheduler.yaml
```
KUBE_SCHEDULER_OPTS="--address=0.0.0.0 \
--terminated-pod-gc-threshold=1\
--kubeconfig=/data/apps/kubernetes/conf/kube-scheduler.kubeconfig \
--leader-elect=true \
--alsologtostderr=true \
--logtostderr=false \
--log-dir=/data/log/kubernetes \
--v=2"
```
# 各种service文件
## kubelet.service
```
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service
[Service]
WorkingDirectory=/data/data/kubelet
ExecStart=/usr/local/bin/kubelet  --bootstrap-kubeconfig=/data/apps/kubernetes/conf/kubelet-bootstrap.kubeconfig   --cert-dir=/data/apps/kubernetes/ssl   --kubeconfig=/data/apps/kubernetes/conf/kubelet.kubeconfig   --config=/data/apps/kubernetes/conf/kubelet.yml   --container-runtime-endpoint=unix:///run/containerd/containerd.sock --v=2 --cert-dir=/data/apps/kubernetes/ssl --container-log-max-files=5 --container-log-max-size=1Gi
Restart=on-failure
RestartSec=5
#StandardOutput=null
[Install]
WantedBy=multi-user.target
```
## kube-apiserver.service
```
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=etcd.service
Wants=etcd.service
[Service]
ExecStart=/usr/local/bin/kube-log-runner -log-file=/data/log/kubernetes/kube-apiserver/kube-apiserver.log --also-stdout=false /usr/local/bin/kube-apiserver --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota --anonymous-auth=false --bind-address=172.16.1.94 --secure-port=6443 --advertise-address=172.16.1.94 --authorization-mode=Node,RBAC --runtime-config=api/all=true --enable-bootstrap-token-auth --service-cluster-ip-range=10.174.0.0/16 --token-auth-file=/data/apps/kubernetes/conf/token.csv --service-node-port-range=30000-50000 --tls-cert-file=/data/apps/kubernetes/ssl/kube-apiserver.pem --tls-private-key-file=/data/apps/kubernetes/ssl/kube-apiserver-key.pem --client-ca-file=/data/apps/kubernetes/ssl/ca.pem --kubelet-client-certificate=/data/apps/kubernetes/ssl/kube-apiserver.pem --kubelet-client-key=/data/apps/kubernetes/ssl/kube-apiserver-key.pem --service-account-key-file=/data/apps/kubernetes/ssl/ca-key.pem --service-account-signing-key-file=/data/apps/kubernetes/ssl/ca-key.pem --service-account-issuer=api --etcd-cafile=/data/apps/kubernetes/ssl/ca.pem --etcd-certfile=/data/apps/kubernetes/ssl/etcd.pem --etcd-keyfile=/data/apps/kubernetes/ssl/etcd-key.pem --etcd-servers=https://172.16.1.94:2379  --allow-privileged=true --apiserver-count=3  --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/data/log/kubernetes/kube-apiserver-audit.log --event-ttl=1h  --v=4 --enable-bootstrap-token-auth  --runtime-config=api/all=true --requestheader-allowed-names=aggregator --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-client-ca-file=/data/apps/kubernetes/ssl/ca.pem --proxy-client-cert-file=/data/apps/kubernetes/ssl/requestheader-proxy-client.pem --proxy-client-key-file=/data/apps/kubernetes/ssl/requestheader-proxy-client-key.pem
Restart=on-failure
RestartSec=5
Type=simple
LimitNOFILE=65536
#StandardOutput=null
[Install]
WantedBy=multi-user.target
```
## kube-controller-manager.service
```
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
#EnvironmentFile=-/data/apps/kubernetes/conf/kube-controller-manager.conf
ExecStart=/usr/local/bin/kube-log-runner -log-file=/data/log/kubernetes/kube-controller-manager/kube-controller-manager.log --also-stdout=false /usr/local/bin/kube-controller-manager  --secure-port=10257  --bind-address=0.0.0.0  --kubeconfig=/data/apps/kubernetes/conf/kube-controller-manager.kubeconfig  --service-cluster-ip-range=10.174.0.0/16  --cluster-name=kubernetes  --cluster-signing-cert-file=/data/apps/kubernetes/ssl/ca.pem  --cluster-signing-key-file=/data/apps/kubernetes/ssl/ca-key.pem   --allocate-node-cidrs=true   --cluster-cidr=10.175.0.0/16   --root-ca-file=/data/apps/kubernetes/ssl/ca.pem   --service-account-private-key-file=/data/apps/kubernetes/ssl/ca-key.pem   --leader-elect=true   --feature-gates=RotateKubeletServerCertificate=true   --controllers=*,bootstrapsigner,tokencleaner   --horizontal-pod-autoscaler-sync-period=10s   --tls-cert-file=/data/apps/kubernetes/ssl/kube-controller-manager.pem   --tls-private-key-file=/data/apps/kubernetes/ssl/kube-controller-manager-key.pem   --use-service-account-credentials=true   --v=4
Restart=on-failure
RestartSec=5
#StandardOutput=null
[Install]
WantedBy=multi-user.target
```
## kube-proxy.service
```
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/kubernetes/kubernetes
After=network.target
[Service]
WorkingDirectory=/data/data/kube-proxy
ExecStart=/usr/local/bin/kube-log-runner -log-file=/data/log/kubernetes/kube-proxy/kube-proxy.log --also-stdout=false  /usr/local/bin/kube-proxy   --config=/data/apps/kubernetes/conf/kube-proxy.yaml  --v=2
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
#StandardOutput=null
[Install]
WantedBy=multi-user.target
```
## kubelet.service
```
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service
[Service]
WorkingDirectory=/data/data/kubelet
ExecStart=/usr/local/bin/kubelet  --bootstrap-kubeconfig=/data/apps/kubernetes/conf/kubelet-bootstrap.kubeconfig   --cert-dir=/data/apps/kubernetes/ssl   --kubeconfig=/data/apps/kubernetes/conf/kubelet.kubeconfig   --config=/data/apps/kubernetes/conf/kubelet.yml   --container-runtime-endpoint=unix:///run/containerd/containerd.sock --v=2 --cert-dir=/data/apps/kubernetes/ssl --container-log-max-files=5 --container-log-max-size=1Gi
Restart=on-failure
RestartSec=5
#StandardOutput=null
[Install]
WantedBy=multi-user.target
```
## etcd.service
```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
  
[Service]
Type=notify
WorkingDirectory=/data/data/etcd/
ExecStart=/usr/local/bin/etcd --config-file=/data/apps/kubernetes/conf/etcd.conf.yml  --peer-client-cert-auth  --client-cert-auth
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
  
[Install]
WantedBy=multi-user.target
```
## /etc/modules-load.d/ipvs.conf
```
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
```
## /etc/modules-load.d/k8s.conf
```
overlay
br_netfilter
```
## /etc/hosts
```
172.16.1.94   k8s-master
172.16.1.129  k8s-node1
172.16.1.23   k8s-node2
172.16.1.220  k8s-node3
```
## /etc/containerd/config.toml
```
disabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/data/data/containerd"
state = "/run/containerd"
temp = ""
version = 2
 
[cgroup]
  path = ""
 
[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0
 
[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0
 
[metrics]
  address = ""
  grpc_histogram = false
 
[plugins]
 
  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"
 
  [plugins."io.containerd.grpc.v1.cri"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_image_defined_volumes = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""
 
    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1
 
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"
 
      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
 
        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]
 
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
 
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
 
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = true
 
      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
 
        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]
 
    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"
 
    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""
 
      [plugins."io.containerd.grpc.v1.cri".registry.auths]
 
      [plugins."io.containerd.grpc.v1.cri".registry.configs]
 
      [plugins."io.containerd.grpc.v1.cri".registry.headers]
 
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://cekcu3pt.mirror.aliyuncs.com"]
 
    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""
 
  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"
 
  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"
 
  [plugins."io.containerd.internal.v1.tracing"]
    sampling_ratio = 1.0
    service_name = "containerd"
 
  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"
 
  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false
 
  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false
 
  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false
 
  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]
 
  [plugins."io.containerd.service.v1.tasks-service"]
    rdt_config_file = ""
 
  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""
 
  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""
 
  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""
 
  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""
 
  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    mount_options = []
    root_path = ""
    sync_remove = false
    upperdir_label = false
 
  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""
 
  [plugins."io.containerd.tracing.processor.v1.otlp"]
    endpoint = ""
    insecure = false
    protocol = ""
 
[proxy_plugins]
 
[stream_processors]
 
  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"
 
  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"
 
[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"
 
[ttrpc]
  address = ""
  gid = 0
  uid = 0
```
生成token：   head -c 16 /dev/urandom| od -An -t x |tr -d ' '  
创建token.csv文件  cat > token.csv << EOF 54e30ab1155a753e192109070466377f,kubelet-bootstrap,10001,"system:kubelet-bootstrap" EOF  
## calico.yaml
```
---
# Source: calico/templates/calico-kube-controllers.yaml
# This manifest creates a Pod Disruption Budget for Controller to allow K8s Cluster Autoscaler to evict
 
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: calico-kube-controllers
  namespace: kube-system
  labels:
    k8s-app: calico-kube-controllers
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      k8s-app: calico-kube-controllers
---
# Source: calico/templates/calico-kube-controllers.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: calico-kube-controllers
  namespace: kube-system
---
# Source: calico/templates/calico-node.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: calico-node
  namespace: kube-system
---
# Source: calico/templates/calico-etcd-secrets.yaml
# The following contains k8s Secrets for use with a TLS enabled etcd cluster.
# For information on populating Secrets, see http://kubernetes.io/docs/user-guide/secrets/
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: calico-etcd-secrets
  namespace: kube-system
data:
  # Populate the following with etcd TLS configuration if desired, but leave blank if
  # not using TLS for etcd.
  # The keys below should be uncommented and the values populated with the base64
  # encoded contents of each file that would be associated with the TLS data.
  # Example command for encoding a file contents: cat <file> | base64 -w 0
  # etcd-key: null
  # etcd-cert: null
  # etcd-ca: null
  etcd-key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBelFMNVhSTkdNL1RHTlRYVkZpTnRPNFhmOGFoNGdJcnRNMTVFdXA1eWNsaTc4cElBCi9HNnJmUFRvSEpoTndnTHA5eHAyQm05RkRCK3BVc3M5dHRyZW80eE1nUkhyUmxNS1hxTm5MK3EzYllZakNMbFoKdXEwWGJ5aXlSS2Z3ZzEzZVovWEJQcjhDWENRcFkrcFBjVUU3R20xNzUvSjBpNVFIWmhBL0Y3MU1jaEZSMU5mTwpZNExxM25FR21qZFZKOUhtZXphUHZpdVdOZFdvdzR3aHhhR1NBay9LZUxRU2J2UHJqbWpJY2JYYlVMdVpMQUpkCmJaZEl3dE5nM0ZJdm96ZW9WanZOSUxTek05WWNxemJyeDNJUHpXRnFTamlPaDJTWHg3T1B0MDNJYmk1ZmN2NFEKdUNRbTdESCt2S2lBcG5oNHQrRzRDVi9lbVgyWlhsbXRYbjlsRFFJREFRQUJBb0lCQVFDemVlMFF2TFR5KzFFaQplRFJLSTAyWGxJWVBLNndDN0p6b0laa052M1QyQWhUWU1WWEhxS05jeTVNQXBaMDlRZ3ZObGs3SkoxUk5YdEovCmR3cGFNSlpFbTZqR1BnZTVFeTI2MkZhWHJtWlM3ZUZ4MjhKZ0dQU3hEZkd6QlVzYjFtdkVtM05JR1RSWnNoYkMKTC9qSWI1RHNlL2pEZ0pEak9QNlpMWlB1bG54OFJuWEZGdUg2YlY5L0ZDRVpYUGx2V1hWc3lESDlkMFNOZmgzaApld21oSzdraytRa1h3cU81d1NlZzdhN1V2T3I4WE1OK085SnRBYnl2RmJMQWY0VklDWUNUM3U2V3NzZ1pLYVNsCnhmMVFlazNCTTBYT3lMblJmaUJoNWtMTXlOSnhsQnFNRzJudXBweEtFaStGQTlHd2NZeU12TnpUcUg0MG5UM0oKVnYxc0czUUJBb0dCQVAveWcvM2FJMlRweXQwL0NRRkRrY2dTUXcwYmlIeXpQVVozSHZQMno0K2wzSjZoWExPeAo3WktiT2xTRTNyS1cxc0MrMWJscVpLQWM3MFA2bDZLdjkzTHVyRFBxems2OExoWU8wWUZ5RndkLzNUYXVmRGQ2ClVIZmZDYk4zZ1FwblpKOTh3ZGNUS1VhVmlqK3JxMjA2c2pvZ29sUHpzV3czdkJaOE5yWERPdGFOQW9HQkFNME4KeG1TSzZUT2FsemplUVVLNndCV0hsR25xNS91eEpmU2dKWHovWjZ1cjFndzg4elhpQzgvSHZqTTc0VzZGOEtsTApmVjBFNXREZmtJWDZTVHhnMVRWUmJvMGNnNmVqUzU2aDJJZGpOMU90SWFhREk1aUlvVnJaNGZZNDF6L0poZWFlCmlBUUZPeFJoTVRpTkQ0QW1EaXF1S0d3NjJqNzFGNDNDVFROZGNHaUJBb0dCQVBDYllqdTgybk1lV1dmOXZ4QmkKSGVTd1Rqby9QT0xGZVFBS01aMzAwcERld25TWml0VWVtaEN0UG50LzRQNlFVRmduempFYzlIV1VYZFZRK1VXbQpHSUFDSVA0NWFUS1pNdFhubmtvTEg5MGI5YkJXL1UwRi9pbUNFZE9WcjBoQmhGVnQ2YWV2U3FraElUTFR4amJMCjdBbzY3WDd3WTBVeGErN1RYSGNvamVKdEFvR0FHblhobVEzWDVBSFo2OHU2YmlyOUtJb1RXOHVsWGZSUktvMFQKNlZwbi9WNHlRK2dGbG5seC9zRU95VHU3N25BNFN4Qmp3QUltNnVNK21odGZJZng0NXVWNE41dHJYZEdUcTRmRgpFa3Q2VTBEdks3YVdmRk45UnVVQTVLNFhFTE1ucFVmbDAyYjlaYmJaREN3ZnlQQ2dPVis1OWFWdWpsdEFTOW03CjdwbnJMSUVDZ1lCWEdlNFVRdFN1RkVEZWd1RXUzcVVBdTBvcWdlM3pZSi9nb09xWEpGa2Nwa1puU2xaYXNsMkEKcEZMcHJtb1NXYmFwemNvZVhoMDZPZlRBKzA0MUF3c2FnV2NsL0NQdlJCdjNhenVSYm41QmdidjI1L2IxdHVvcwo5TjExM2NOYXE2cVRzZ3E1UWhqQ21DSU12VjZWQSt6V1g0Wmt1Z1lUaGNnNzloSjhUemNObFE9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
  etcd-cert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUVEakNDQXZhZ0F3SUJBZ0lVWkhCb0hQQ0s1TmU3a24zV0Q0OW90WUpmalhZd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1pURUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjFOcFkyaDFZVzR4RURBT0JnTlZCQWNUQjJObwpaVzVuWkhVeEREQUtCZ05WQkFvVEEyczRjekVQTUEwR0ExVUVDeE1HYzNsemRHVnRNUk13RVFZRFZRUURFd3ByCmRXSmxjbTVsZEdWek1CNFhEVEl6TVRFeU16QTNOVFl3TUZvWERUTXpNVEV5TURBM05UWXdNRm93WHpFTE1Ba0cKQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjFOcFkyaDFZVzR4RURBT0JnTlZCQWNUQjJOb1pXNW5aSFV4RERBSwpCZ05WQkFvVEEyczRjekVQTUEwR0ExVUVDeE1HYzNsemRHVnRNUTB3Q3dZRFZRUURFd1JsZEdOa01JSUJJakFOCkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXpRTDVYUk5HTS9UR05UWFZGaU50TzRYZjhhaDQKZ0lydE0xNUV1cDV5Y2xpNzhwSUEvRzZyZlBUb0hKaE53Z0xwOXhwMkJtOUZEQitwVXNzOXR0cmVvNHhNZ1JIcgpSbE1LWHFObkwrcTNiWVlqQ0xsWnVxMFhieWl5Uktmd2cxM2VaL1hCUHI4Q1hDUXBZK3BQY1VFN0dtMTc1L0owCmk1UUhaaEEvRjcxTWNoRlIxTmZPWTRMcTNuRUdtamRWSjlIbWV6YVB2aXVXTmRXb3c0d2h4YUdTQWsvS2VMUVMKYnZQcmptakljYlhiVUx1WkxBSmRiWmRJd3ROZzNGSXZvemVvVmp2TklMU3pNOVljcXpicngzSVB6V0ZxU2ppTwpoMlNYeDdPUHQwM0liaTVmY3Y0UXVDUW03REgrdktpQXBuaDR0K0c0Q1YvZW1YMlpYbG10WG45bERRSURBUUFCCm80RzdNSUc0TUE0R0ExVWREd0VCL3dRRUF3SUZvREFkQmdOVkhTVUVGakFVQmdnckJnRUZCUWNEQVFZSUt3WUIKQlFVSEF3SXdEQVlEVlIwVEFRSC9CQUl3QURBZEJnTlZIUTRFRmdRVW0vUlk5VFVJeUE3ZDFXNTdYZ0s2WXNXdQpGNTB3SHdZRFZSMGpCQmd3Rm9BVXo1RXRoRUZDdmRaYTE2dnc2Z0YrOHhFV092Z3dPUVlEVlIwUkJESXdNSWNFCmZ3QUFBWWNFckJBNFhvY0VyQkE0M0ljRXJCQTRGNGNFckJBNGdZY0VyQkE0NTRjRXJCQTQ1b2NFckJBNDVUQU4KQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBQ3BPRnpDbHNCMWR5VDlEdENuRHNaSWdVNDVoWmcrR0pIWmNYaTFycgpUeVVmbkVaZUVvNmFobkZSeTh0cmdoNTBJcnNTSFZUd1JRK2R0aEVFRFBLdUFDR1R5SEpZM1FKQ2JYWU9WSGNnCmRVWld5S3ZzTXM5RUVtNXlzbzJsa3hBWFhLMXFvemc2blYrblhUOXMwcnBodFNEVmRJaUkrZkxLUnVHcDNoOWkKUzVFYW1JaFE4QkF2RUdnRDVsMHB2enQ4QTdUZHoyVExDM0U3TkM0MFo0UnVIY0FYMDBoejgzNEVjNy9vamZ1NApTUHBHVGpBa3k3SXJmRzZDaW5YY0tha2s5SVNHMUpJUmZwdG13V1c4SFN5a25vQ1RyMUlTQ3pIQnFrQW1FUDBSCkFmcnBScXhDcG02NDlBMzBqT3BGQ0wzdFl0WlBRbGtYa2M2REJ2VHVEem9GblE9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
  etcd-ca: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURtakNDQW9LZ0F3SUJBZ0lVYTlLUGwrRi9JSmViZFBCNzFnN0FuR1ZBQjBZd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1pURUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjFOcFkyaDFZVzR4RURBT0JnTlZCQWNUQjJObwpaVzVuWkhVeEREQUtCZ05WQkFvVEEyczRjekVQTUEwR0ExVUVDeE1HYzNsemRHVnRNUk13RVFZRFZRUURFd3ByCmRXSmxjbTVsZEdWek1CNFhEVEl6TVRFeU16QTNOVFl3TUZvWERUTXpNVEV5TURBM05UWXdNRm93WlRFTE1Ba0cKQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjFOcFkyaDFZVzR4RURBT0JnTlZCQWNUQjJOb1pXNW5aSFV4RERBSwpCZ05WQkFvVEEyczRjekVQTUEwR0ExVUVDeE1HYzNsemRHVnRNUk13RVFZRFZRUURFd3ByZFdKbGNtNWxkR1Z6Ck1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBcTA5SjhDSDd3RjVlVHpWVkZvREUKRVBzbGhGcEczL3I3dkl0aHZxU09wTFlUajZuQzBzYzJDaERHZEVYQ3BObVJzd2J6OGUyQVA2SWlFS3F0QXB5dQpwaXBzSDJ0WEhMRUpYOFllV1Q5WlJTNzEwdTM3VUtjRnhJNWFobnpORHFVR0hQUHMxbHNIVktmUEN4VXpVQThTCkdYZjRFaFFEdUZsblgrelJCdjE3Lyt1UzlsZFpla014R3B0RUZ5YmxVSXRCeG5nS0ViazRHS2RQeUZGeERGM1UKNkR0WlNQL2M0WXRUSkdFOHZ0aURIWHlUWmc0TGF4REVNQVZVdFFIUTU5M3N4Y0twdXVtSEVPMmY2R1RmVFlVZQpOcWVEYUg2SzB5Z0o3M0ZET2cyd1AxTlhBMit3TUJLT1dmZGVWNmlJcXNtcHdLUUVNamFtclEyN0FjZFE0Y1pvCjFRSURBUUFCbzBJd1FEQU9CZ05WSFE4QkFmOEVCQU1DQVFZd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlYKSFE0RUZnUVV6NUV0aEVGQ3ZkWmExNnZ3NmdGKzh4RVdPdmd3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUh4QwpxSTBvcGZkcEJMNUZuZXRxcFhqNGljSlRoRU5ZMnd3dWVnclVRbjBIRVRHMnZIY0JRQWxYMXpFUVNjTllOS0NsClBEaHErWmY5SFRQcjMvNUZ2eEREN1NKZFNuNmU3dmUxZmFZdkJRbm52T01nc05lK2lCWko0eS9TbHBuWk9FVkwKRkNOMjd1QmloN3dhcTA3NmhIVlQ1K1dwNll6a2pIT0ZsVFU4b1BjeXQwOHI4QUROM1VNVWlGcHdkTXgxK2wxRQpMSiswT0tvc0FyNU80Z3J6TERudFZrQXN4bWtuMG5pRlZMZTRic0ZrOE8ra2FYN0ZSUEt0SkVRMWc2RjdVeUpvCkVSVTVSZi9tdkJiN3ZyK1AwY21DaXdBa0ppcWpVWGY4M1ZJTzJGUUVHeFNtSU05YjFVZTNDbjZoUUwrTENpMjAKZVRreEZZM3hWbjRxQkZMSVBJaz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
---
# Source: calico/templates/calico-config.yaml
# This ConfigMap is used to configure a self-hosted Calico installation.
kind: ConfigMap
apiVersion: v1
metadata:
  name: calico-config
  namespace: kube-system
data:
  # Configure this with the location of your etcd cluster.
  etcd_endpoints: "https://172.16.1.94:2379"
  # If you're using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  etcd_ca: "/calico-secrets/etcd-ca"   # "/calico-secrets/etcd-ca"
  etcd_cert: "/calico-secrets/etcd-cert" # "/calico-secrets/etcd-cert"
  etcd_key: "/calico-secrets/etcd-key"  # "/calico-secrets/etcd-key"
  # Typha is disabled.
  typha_service_name: "none"
  # Configure the backend to use.
  calico_backend: "vxlan"
  #calico_backend: "bird"
 
  # Configure the MTU to use for workload interfaces and tunnels.
  # By default, MTU is auto-detected, and explicitly setting this field should not be required.
  # You can override auto-detection by providing a non-zero value.
  veth_mtu: "0"
 
  # The CNI network configuration to install on each node. The special
  # values in this config will be automatically populated.
  cni_network_config: |-
    {
      "name": "k8s-pod-network",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "calico",
          "log_level": "info",
          "log_file_path": "/var/log/calico/cni/cni.log",
          "etcd_endpoints": "__ETCD_ENDPOINTS__",
          "etcd_key_file": "__ETCD_KEY_FILE__",
          "etcd_cert_file": "__ETCD_CERT_FILE__",
          "etcd_ca_cert_file": "__ETCD_CA_CERT_FILE__",
          "mtu": __CNI_MTU__,
          "ipam": {
              "type": "calico-ipam"
          },
          "policy": {
              "type": "k8s"
          },
          "kubernetes": {
              "kubeconfig": "__KUBECONFIG_FILEPATH__"
          }
        },
        {
          "type": "portmap",
          "snat": true,
          "capabilities": {"portMappings": true}
        },
        {
          "type": "bandwidth",
          "capabilities": {"bandwidth": true}
        }
      ]
    }
---
# Source: calico/templates/calico-kube-controllers-rbac.yaml
# Include a clusterrole for the kube-controllers component,
# and bind it to the calico-kube-controllers serviceaccount.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: calico-kube-controllers
rules:
  # Pods are monitored for changing labels.
  # The node controller monitors Kubernetes nodes.
  # Namespace and serviceaccount labels are used for policy.
  - apiGroups: [""]
    resources:
      - pods
      - nodes
      - namespaces
      - serviceaccounts
    verbs:
      - watch
      - list
      - get
  # Watch for changes to Kubernetes NetworkPolicies.
  - apiGroups: ["networking.k8s.io"]
    resources:
      - networkpolicies
    verbs:
      - watch
      - list
---
# Source: calico/templates/calico-node-rbac.yaml
# Include a clusterrole for the calico-node DaemonSet,
# and bind it to the calico-node serviceaccount.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: calico-node
rules:
  # Used for creating service account tokens to be used by the CNI plugin
  - apiGroups: [""]
    resources:
      - serviceaccounts/token
    resourceNames:
      - calico-node
    verbs:
      - create
  # The CNI plugin needs to get pods, nodes, and namespaces.
  - apiGroups: [""]
    resources:
      - pods
      - nodes
      - namespaces
    verbs:
      - get
  # EndpointSlices are used for Service-based network policy rule
  # enforcement.
  - apiGroups: ["discovery.k8s.io"]
    resources:
      - endpointslices
    verbs:
      - watch
      - list
  - apiGroups: [""]
    resources:
      - endpoints
      - services
    verbs:
      # Used to discover service IPs for advertisement.
      - watch
      - list
  # Pod CIDR auto-detection on kubeadm needs access to config maps.
  - apiGroups: [""]
    resources:
      - configmaps
    verbs:
      - get
  - apiGroups: [""]
    resources:
      - nodes/status
    verbs:
      # Needed for clearing NodeNetworkUnavailable flag.
      - patch
---
# Source: calico/templates/calico-kube-controllers-rbac.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: calico-kube-controllers
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: calico-kube-controllers
subjects:
- kind: ServiceAccount
  name: calico-kube-controllers
  namespace: kube-system
---
# Source: calico/templates/calico-node-rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: calico-node
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: calico-node
subjects:
- kind: ServiceAccount
  name: calico-node
  namespace: kube-system
---
# Source: calico/templates/calico-node.yaml
# This manifest installs the calico-node container, as well
# as the CNI plugins and network config on
# each master and worker node in a Kubernetes cluster.
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: calico-node
  namespace: kube-system
  labels:
    k8s-app: calico-node
spec:
  selector:
    matchLabels:
      k8s-app: calico-node
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        k8s-app: calico-node
    spec:
      nodeSelector:
        kubernetes.io/os: linux
      hostNetwork: true
      tolerations:
        # Make sure calico-node gets scheduled on all nodes.
        - effect: NoSchedule
          operator: Exists
        # Mark the pod as a critical add-on for rescheduling.
        - key: CriticalAddonsOnly
          operator: Exists
        - effect: NoExecute
          operator: Exists
      serviceAccountName: calico-node
      # Minimize downtime during a rolling upgrade or deletion; tell Kubernetes to do a "force
      # deletion": https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods.
      terminationGracePeriodSeconds: 0
      priorityClassName: system-node-critical
      initContainers:
        # This container installs the CNI binaries
        # and CNI network config file on each node.
        - name: install-cni
          image: docker.io/calico/cni:v3.25.2
          imagePullPolicy: IfNotPresent
          command: ["/opt/cni/bin/install"]
          envFrom:
          - configMapRef:
              # Allow KUBERNETES_SERVICE_HOST and KUBERNETES_SERVICE_PORT to be overridden for eBPF mode.
              name: kubernetes-services-endpoint
              optional: true
          env:
            # Name of the CNI config file to create.
            - name: CNI_CONF_NAME
              value: "10-calico.conflist"
            # The CNI network config to install on each node.
            - name: CNI_NETWORK_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: cni_network_config
            # The location of the etcd cluster.
            - name: ETCD_ENDPOINTS
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_endpoints
            # CNI MTU Config variable
            - name: CNI_MTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # Prevents the container from sleeping forever.
            - name: SLEEP
              value: "false"
          volumeMounts:
            - mountPath: /host/opt/cni/bin
              name: cni-bin-dir
            - mountPath: /host/etc/cni/net.d
              name: cni-net-dir
            - mountPath: /calico-secrets
              name: etcd-certs
          securityContext:
            privileged: true
        # This init container mounts the necessary filesystems needed by the BPF data plane
        # i.e. bpf at /sys/fs/bpf and cgroup2 at /run/calico/cgroup. Calico-node initialisation is executed
        # in best effort fashion, i.e. no failure for errors, to not disrupt pod creation in iptable mode.
        - name: "mount-bpffs"
          image: docker.io/calico/node:v3.25.2
          imagePullPolicy: IfNotPresent
          command: ["calico-node", "-init", "-best-effort"]
          volumeMounts:
            - mountPath: /sys/fs
              name: sys-fs
              # Bidirectional is required to ensure that the new mount we make at /sys/fs/bpf propagates to the host
              # so that it outlives the init container.
              mountPropagation: Bidirectional
            - mountPath: /var/run/calico
              name: var-run-calico
              # Bidirectional is required to ensure that the new mount we make at /run/calico/cgroup propagates to the host
              # so that it outlives the init container.
              mountPropagation: Bidirectional
            # Mount /proc/ from host which usually is an init program at /nodeproc. It's needed by mountns binary,
            # executed by calico-node, to mount root cgroup2 fs at /run/calico/cgroup to attach CTLB programs correctly.
            - mountPath: /nodeproc
              name: nodeproc
              readOnly: true
          securityContext:
            privileged: true
      containers:
        # Runs calico-node container on each Kubernetes node. This
        # container programs network policy and routes on each
        # host.
        - name: calico-node
          image: docker.io/calico/node:v3.25.2
          imagePullPolicy: IfNotPresent
          envFrom:
          - configMapRef:
              # Allow KUBERNETES_SERVICE_HOST and KUBERNETES_SERVICE_PORT to be overridden for eBPF mode.
              name: kubernetes-services-endpoint
              optional: true
          env:
            # The location of the etcd cluster.
            - name: ETCD_ENDPOINTS
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_endpoints
            # Location of the CA certificate for etcd.
            - name: ETCD_CA_CERT_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_ca
            # Location of the client key for etcd.
            - name: ETCD_KEY_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_key
            # Location of the client certificate for etcd.
            - name: ETCD_CERT_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_cert
            # Set noderef for node controller.
            - name: CALICO_K8S_NODE_REF
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            # Choose the backend to use.
            - name: CALICO_NETWORKING_BACKEND
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: calico_backend
            # Cluster type to identify the deployment type
            - name: CLUSTER_TYPE
              value: "k8s,bgp"
            # Auto-detect the BGP IP address.
            - name: IP
              value: "autodetect"
            # Enable IPIP
            - name: CALICO_IPV4POOL_IPIP
              value: "Never"
            # Enable or Disable VXLAN on the default IP pool.
            - name: CALICO_IPV4POOL_VXLAN
              value: "Always"
            # Enable or Disable VXLAN on the default IPv6 IP pool.
            - name: CALICO_IPV6POOL_VXLAN
              value: "Never"
            # Set MTU for tunnel device used if ipip is enabled
            - name: FELIX_IPINIPMTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # Set MTU for the VXLAN tunnel device.
            - name: FELIX_VXLANMTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # Set MTU for the Wireguard tunnel device.
            - name: FELIX_WIREGUARDMTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            - name: IP_AUTODETECTION_METHOD
              value: "interface=ens.*"
            # The default IPv4 pool to create on startup if none exists. Pod IPs will be
            # chosen from this range. Changing this value after installation will have
            # no effect. This should fall within `--cluster-cidr`.
            # - name: CALICO_IPV4POOL_CIDR
            #   value: "192.168.0.0/16"
            - name: CALICO_IPV4POOL_CIDR
              value: "10.175.0.0/16"
            # Disable file logging so `kubectl logs` works.
            - name: CALICO_DISABLE_FILE_LOGGING
              value: "true"
            # Set Felix endpoint to host default action to ACCEPT.
            - name: FELIX_DEFAULTENDPOINTTOHOSTACTION
              value: "ACCEPT"
            # Disable IPv6 on Kubernetes.
            - name: FELIX_IPV6SUPPORT
              value: "false"
            - name: FELIX_HEALTHENABLED
              value: "true"
          securityContext:
            privileged: true
          resources:
            requests:
              cpu: 250m
          lifecycle:
            preStop:
              exec:
                command:
                - /bin/calico-node
                - -shutdown
          livenessProbe:
            exec:
              command:
              - /bin/calico-node
              - -felix-live
                # - -bird-live
            periodSeconds: 10
            initialDelaySeconds: 10
            failureThreshold: 6
            timeoutSeconds: 10
          readinessProbe:
            exec:
              command:
              - /bin/calico-node
              - -felix-ready
                # - -bird-ready
            periodSeconds: 10
            timeoutSeconds: 10
          volumeMounts:
            # For maintaining CNI plugin API credentials.
            - mountPath: /host/etc/cni/net.d
              name: cni-net-dir
              readOnly: false
            - mountPath: /lib/modules
              name: lib-modules
              readOnly: true
            - mountPath: /run/xtables.lock
              name: xtables-lock
              readOnly: false
            - mountPath: /var/run/calico
              name: var-run-calico
              readOnly: false
            - mountPath: /var/lib/calico
              name: var-lib-calico
              readOnly: false
            - mountPath: /calico-secrets
              name: etcd-certs
            - name: policysync
              mountPath: /var/run/nodeagent
            # For eBPF mode, we need to be able to mount the BPF filesystem at /sys/fs/bpf so we mount in the
            # parent directory.
            - name: bpffs
              mountPath: /sys/fs/bpf
            - name: cni-log-dir
              mountPath: /var/log/calico/cni
              readOnly: true
      volumes:
        # Used by calico-node.
        - name: lib-modules
          hostPath:
            path: /lib/modules
        - name: var-run-calico
          hostPath:
            path: /var/run/calico
        - name: var-lib-calico
          hostPath:
            path: /var/lib/calico
        - name: xtables-lock
          hostPath:
            path: /run/xtables.lock
            type: FileOrCreate
        - name: sys-fs
          hostPath:
            path: /sys/fs/
            type: DirectoryOrCreate
        - name: bpffs
          hostPath:
            path: /sys/fs/bpf
            type: Directory
        # mount /proc at /nodeproc to be used by mount-bpffs initContainer to mount root cgroup2 fs.
        - name: nodeproc
          hostPath:
            path: /proc
        # Used to install CNI.
        - name: cni-bin-dir
          hostPath:
            path: /opt/cni/bin
        - name: cni-net-dir
          hostPath:
            path: /etc/cni/net.d
        # Used to access CNI logs.
        - name: cni-log-dir
          hostPath:
            path: /var/log/calico/cni
        # Mount in the etcd TLS secrets with mode 400.
        # See https://kubernetes.io/docs/concepts/configuration/secret/
        - name: etcd-certs
          secret:
            secretName: calico-etcd-secrets
            defaultMode: 0400
        # Used to create per-pod Unix Domain Sockets
        - name: policysync
          hostPath:
            type: DirectoryOrCreate
            path: /var/run/nodeagent
---
# Source: calico/templates/calico-kube-controllers.yaml
# See https://github.com/projectcalico/kube-controllers
apiVersion: apps/v1
kind: Deployment
metadata:
  name: calico-kube-controllers
  namespace: kube-system
  labels:
    k8s-app: calico-kube-controllers
spec:
  # The controllers can only have a single active instance.
  replicas: 1
  selector:
    matchLabels:
      k8s-app: calico-kube-controllers
  strategy:
    type: Recreate
  template:
    metadata:
      name: calico-kube-controllers
      namespace: kube-system
      labels:
        k8s-app: calico-kube-controllers
    spec:
      nodeSelector:
        kubernetes.io/os: linux
      tolerations:
        # Mark the pod as a critical add-on for rescheduling.
        - key: CriticalAddonsOnly
          operator: Exists
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
        - key: node-role.kubernetes.io/control-plane
          effect: NoSchedule
      serviceAccountName: calico-kube-controllers
      priorityClassName: system-cluster-critical
      # The controllers must run in the host network namespace so that
      # it isn't governed by policy that would prevent it from working.
      hostNetwork: true
      containers:
        - name: calico-kube-controllers
          image: docker.io/calico/kube-controllers:v3.25.2
          imagePullPolicy: IfNotPresent
          env:
            # The location of the etcd cluster.
            - name: ETCD_ENDPOINTS
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_endpoints
            # Location of the CA certificate for etcd.
            - name: ETCD_CA_CERT_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_ca
            # Location of the client key for etcd.
            - name: ETCD_KEY_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_key
            # Location of the client certificate for etcd.
            - name: ETCD_CERT_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_cert
            # Choose which controllers to run.
            - name: ENABLED_CONTROLLERS
              value: policy,namespace,serviceaccount,workloadendpoint,node
          volumeMounts:
            # Mount in the etcd TLS secrets.
            - mountPath: /calico-secrets
              name: etcd-certs
          livenessProbe:
            exec:
              command:
              - /usr/bin/check-status
              - -l
            periodSeconds: 10
            initialDelaySeconds: 10
            failureThreshold: 6
            timeoutSeconds: 10
          readinessProbe:
            exec:
              command:
              - /usr/bin/check-status
              - -r
            periodSeconds: 10
      volumes:
        # Mount in the etcd TLS secrets with mode 400.
        # See https://kubernetes.io/docs/concepts/configuration/secret/
        - name: etcd-certs
          secret:
            secretName: calico-etcd-secrets
            defaultMode: 0440
coredns.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: coredns
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:coredns
rules:
  - apiGroups:
    - ""
    resources:
    - endpoints
    - services
    - pods
    - namespaces
    verbs:
    - list
    - watch
  - apiGroups:
    - discovery.k8s.io
    resources:
    - endpointslices
    verbs:
    - list
    - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:coredns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:coredns
subjects:
- kind: ServiceAccount
  name: coredns
  namespace: kube-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
          lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf {
          max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coredns
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/name: "CoreDNS"
    app.kubernetes.io/name: coredns
spec:
  # replicas: not specified here:
  # 1. Default is 1.
  # 2. Will be tuned in real time if DNS horizontal auto-scaling is turned on.
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  selector:
    matchLabels:
      k8s-app: kube-dns
      app.kubernetes.io/name: coredns
  template:
    metadata:
      labels:
        k8s-app: kube-dns
        app.kubernetes.io/name: coredns
    spec:
      priorityClassName: system-cluster-critical
      serviceAccountName: coredns
      tolerations:
        - key: "CriticalAddonsOnly"
          operator: "Exists"
      nodeSelector:
        kubernetes.io/os: linux
      affinity:
         podAntiAffinity:
           requiredDuringSchedulingIgnoredDuringExecution:
           - labelSelector:
               matchExpressions:
               - key: k8s-app
                 operator: In
                 values: ["kube-dns"]
             topologyKey: kubernetes.io/hostname
      containers:
      - name: coredns
        image: coredns/coredns:1.9.4
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            memory: 170Mi
          requests:
            cpu: 100m
            memory: 70Mi
        args: [ "-conf", "/etc/coredns/Corefile" ]
        volumeMounts:
        - name: config-volume
          mountPath: /etc/coredns
          readOnly: true
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
        - containerPort: 9153
          name: metrics
          protocol: TCP
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - all
          readOnlyRootFilesystem: true
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 8181
            scheme: HTTP
      dnsPolicy: Default
      volumes:
        - name: config-volume
          configMap:
            name: coredns
            items:
            - key: Corefile
              path: Corefile
---
apiVersion: v1
kind: Service
metadata:
  name: kube-dns
  namespace: kube-system
  annotations:
    prometheus.io/port: "9153"
    prometheus.io/scrape: "true"
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: "CoreDNS"
    app.kubernetes.io/name: coredns
spec:
  selector:
    k8s-app: kube-dns
    app.kubernetes.io/name: coredns
  clusterIP: 10.174.0.2
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
    protocol: TCP
  - name: metrics
    port: 9153
    protocol: TCP
```
crictl.yaml
```
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: true
pull-image-on-create: false
```
## metric-server.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      hostNetwork: true
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                k8s-app: metrics-server
            namespaces:
            - kube-system
            topologyKey: kubernetes.io/hostname
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --kubelet-insecure-tls
        - --metric-resolution=15s
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.6.4
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 4443
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: metrics-server
  namespace: kube-system
spec:
  minAvailable: 1
  selector:
    matchLabels:
      k8s-app: metrics-server
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```
## node-export.yaml
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: crane-system
  labels:
    k8s-app: node-exporter
spec:
  selector:
    matchLabels:
        k8s-app: node-exporter
  template:
    metadata:
      labels:
        k8s-app: node-exporter
    spec:
      tolerations:    #污点配置,不往master上调度这个
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
      containers:
      - image: quay.io/prometheus/node-exporter:latest
        imagePullPolicy: IfNotPresent
        name: prometheus-node-exporter
        ports:
        - containerPort: 9100
          hostPort: 9100
          protocol: TCP
          name: metrics
        volumeMounts:
        - mountPath: /host/proc
          name: proc
        - mountPath: /host/sys
          name: sys
        - mountPath: /host
          name: rootfs
        args:
        - --path.procfs=/host/proc
        - --path.sysfs=/host/sys
        - --path.rootfs=/host
      volumes:
        - name: proc
          hostPath:
            path: /proc
        - name: sys
          hostPath:
            path: /sys
        - name: rootfs
          hostPath:
            path: /
      hostNetwork: true
      hostPID: true
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    prometheus.io/scrape: "true"
  labels:
    k8s-app: node-exporter
  name: node-exporter
  namespace: crane-system
spec:
  ports:
  - name: http
    port: 9100
    protocol: TCP
  selector:
    k8s-app: node-exporter
 
---
# prom.rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: crane-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  - services
  - endpoints
  - pods
  - nodes/proxy
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - "extensions"
  resources:
    - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - configmaps
  - nodes/metrics
  verbs:
  - get
- nonResourceURLs:
  - /metrics
  verbs:
  - get
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: crane-system
---
apiVersion: v1
kind: Secret
metadata:
  name: prometheus
  namespace: crane-system
  annotations:
    kubernetes.io/service-account.name: prometheus
type: kubernetes.io/service-account-token
```
## limits.conf
```
* soft nofile  100001
* hard nofile  100002
root soft nofile 100001
root hard nofile 100002
* hard core 0
# End of file
```
## admin-csr.json
```
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Sichuan",
      "L": "chengdu",
      "O": "system:masters",            
      "OU": "system"
    }
  ]
}
```
## ca-config.json
```
{
  "signing": {
      "default": {
          "expiry": "87600h"
        },
      "profiles": {
          "kubernetes": {
              "usages": [
                  "signing",
                  "key encipherment",
                  "server auth",
                  "client auth"
              ],
              "expiry": "87600h"
          }
      }
  }
}
```
## ca-csr.json
```
{
  "CN": "kubernetes",
  "key": {
      "algo": "rsa",
      "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Sichuan",
      "L": "chengdu",
      "O": "k8s",
      "OU": "system"
    }
  ],
  "ca": {
          "expiry": "87600h"
  }
}
```
## datacenter-dev-user.json
```
{
  "CN": "datacenter-dev-cd-user",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Sichuan",
      "L": "chengdu",
      "O": "k8s",            
      "OU": "system"
    }
  ]
}
```
## etcd-apisix-csr.json 
```
{
  "CN": "etcd-apisix",
  "hosts": [
    "127.0.0.1",
    "172.16.1.94",
    "172.16.1.220",
    "172.16.1.23",
    "172.16.1.129",
    "172.16.1.231",
    "172.16.1.230",
    "172.16.1.229"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [{
    "C": "CN",
    "ST": "Sichuan",
    "L": "chengdu",
    "O": "k8s",
    "OU": "system"
  }]
}
```
## etcd-csr.json
```
{
  "CN": "etcd",
  "hosts": [
    "127.0.0.1",
    "172.16.1.94",
    "172.16.1.220",
    "172.16.1.23",
    "172.16.1.129",
    "172.16.1.231",
    "172.16.1.230",
    "172.16.1.229"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [{
    "C": "CN",
    "ST": "Sichuan",
    "L": "chengdu",
    "O": "k8s",
    "OU": "system"
  }]
}
```
## kube-apiserver-csr.json
```
{
"CN": "kubernetes",
  "hosts": [
    "127.0.0.1",
    "172.16.1.94",
    "172.16.1.220",
    "172.16.1.23",
    "172.16.1.129",
    "172.16.1.231",
    "172.16.1.230",
    "172.16.1.229",
    "10.174.0.1",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Sichuan",
      "L": "chengdu",
      "O": "k8s",
      "OU": "system"
    }
  ]
}
```
## kube-controller-manager-csr.json
```
{
    "CN": "system:kube-controller-manager",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "hosts": [
      "127.0.0.1",
      "172.16.1.94"
    ],
    "names": [
      {
        "C": "CN",
        "ST": "Sichuan",
        "L": "chengdu",
        "O": "system:kube-controller-manager",
        "OU": "system"
      }
    ]
}
```
## kube-proxy-csr.json
```
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Sichuan",
      "L": "chengdu",
      "O": "k8s",
      "OU": "system"
    }
  ]
}
```
## kube-scheduler-csr.json
```
{
    "CN": "system:kube-scheduler",
    "hosts": [
      "127.0.0.1",
      "172.16.1.94"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "Sichuan",
        "L": "chengdu",
        "O": "system:kube-scheduler",
        "OU": "system"
      }
    ]
}
```
生成配置
```
# 生成ca
cfssl gencert -initca ca-csr.json  | cfssljson -bare ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes etcd-csr.json | cfssljson  -bare etcd
# 生成token
cat > token.csv << EOF
$(head -c 16 /dev/urandom | od -An -t x | tr -d ' '),kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF
# kube-apiserver
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-apiserver-csr.json | cfssljson -bare kube-apiserver
# 客户端证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.7.10:6443 --kubeconfig=kube.config
kubectl config set-credentials admin --client-certificate=admin.pem --client-key=admin-key.pem --embed-certs=true --kubeconfig=kube.config
kubectl config set-context kubernetes --cluster=kubernetes --user=admin --kubeconfig=kube.config
kubectl config use-context kubernetes --kubeconfig=kube.config
kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user kubernetes
# kube-controller-manager
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.7.10:6443 --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-credentials system:kube-controller-manager --client-certificate=kube-controller-manager.pem --client-key=kube-controller-manager-key.pem --embed-certs=true --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-context system:kube-controller-manager --cluster=kubernetes --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
kubectl config use-context system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
# kube-scheduler
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.7.10:6443 --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-credentials system:kube-scheduler --client-certificate=kube-scheduler.pem --client-key=kube-scheduler-key.pem --embed-certs=true --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-context system:kube-scheduler --cluster=kubernetes --user=system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
kubectl config use-context system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
# kubelet
BOOTSTRAP_TOKEN=$(awk -F "," '{print $1}' /etc/kubernetes/token.csv)
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.7.10:6443 --kubeconfig=kubelet-bootstrap.kubeconfig
kubectl config set-credentials kubelet-bootstrap --token=${BOOTSTRAP_TOKEN} --kubeconfig=kubelet-bootstrap.kubeconfig
kubectl config set-context default --cluster=kubernetes --user=kubelet-bootstrap --kubeconfig=kubelet-bootstrap.kubeconfig
kubectl config use-context default --kubeconfig=kubelet-bootstrap.kubeconfig
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap
# kube-proxy
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://192.168.7.10:6443 --kubeconfig=kube-proxy.kubeconfig
kubectl config set-credentials kube-proxy --client-certificate=kube-proxy.pem --client-key=kube-proxy-key.pem --embed-certs=true --kubeconfig=kube-proxy.kubeconfig
kubectl config set-context default --cluster=kubernetes --user=kube-proxy --kubeconfig=kube-proxy.kubeconfig
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

参考：

https://www.cnblogs.com/fengdejiyixx/p/16576021.html


