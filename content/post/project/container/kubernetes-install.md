---
title: "Kubernetes Install"
date: 2023-11-23T16:19:38+08:00
categories:
- 项目
- kubernetes相关
tags:
- kubernetes
- 记录
keywords:
- 安装
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

```
/usr/local/bin/kube-apiserver --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota --anonymous-auth=false --bind-address=192.168.1.1 --secure-port=6443 --advertise-address=192.168.1.1 --authorization-mode=Node,RBAC --runtime-config=api/all=true --enable-bootstrap-token-auth --service-cluster-ip-range=10.174.0.0/16 --token-auth-file=/data/apps/kubernetes/conf/token.csv --service-node-port-range=30000-50000 --tls-cert-file=/data/apps/kubernetes/ssl/kube-apiserver.pem --tls-private-key-file=/data/apps/kubernetes/ssl/kube-apiserver-key.pem --client-ca-file=/data/apps/kubernetes/ssl/ca.pem --kubelet-client-certificate=/data/apps/kubernetes/ssl/kube-apiserver.pem --kubelet-client-key=/data/apps/kubernetes/ssl/kube-apiserver-key.pem --service-account-key-file=/data/apps/kubernetes/ssl/ca-key.pem --service-account-signing-key-file=/data/apps/kubernetes/ssl/ca-key.pem --service-account-issuer=api --etcd-cafile=/data/apps/kubernetes/ssl/ca.pem --etcd-certfile=/data/apps/kubernetes/ssl/etcd.pem --etcd-keyfile=/data/apps/kubernetes/ssl/etcd-key.pem --etcd-servers=https://192.168.1.1:2379  --allow-privileged=true --apiserver-count=3  --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/data/log/kubernetes/kube-apiserver-audit.log --event-ttl=1h  --v=4
```
```
cfssl gencert -initca ca-csr.json  | cfssljson -bare ca
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes etcd-csr.json | cfssljson  -bare etcd
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-apiserver-csr.json | cfssljson -bare kube-apiserver
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes admin-csr.json | cfssljson -bare admin
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
```
```
cat > token.csv << EOF
$(head -c 16 /dev/urandom | od -An -t x | tr -d ' '),kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF
```
```
kubectl config set-cluster kubernetes --certificate-authority=../ssl/ca.pem --embed-certs=true --server=https://192.168.1.1:6443 --kubeconfig=kube.config
kubectl config set-credentials admin --client-certificate=../ssl/admin.pem --client-key=../ssl/admin-key.pem --embed-certs=true --kubeconfig=kube.config
kubectl config set-context kubernetes --cluster=kubernetes --user=admin --kubeconfig=kube.config
kubectl config use-context kubernetes --kubeconfig=kube.config
mkdir ~/.kube -p 
cp kube.config ~/.kube/config
cp kube.config /data/apps/kubernetes/admin.conf
kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user kubernetes
```
```
kubectl config set-cluster kubernetes --certificate-authority=../ssl/ca.pem --embed-certs=true --server=https://192.168.1.1:6443 --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-credentials system:kube-controller-manager --client-certificate=../ssl/kube-controller-manager.pem --client-key=../ssl/kube-controller-manager-key.pem --embed-certs=true --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-context system:kube-controller-manager --cluster=kubernetes --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
kubectl config use-context system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
```
```
kubectl config set-cluster kubernetes --certificate-authority=../ssl/ca.pem --embed-certs=true --server=https://192.168.1.1:6443 --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-credentials system:kube-scheduler --client-certificate=../ssl/kube-scheduler.pem --client-key=../ssl/kube-scheduler-key.pem --embed-certs=true --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-context system:kube-scheduler --cluster=kubernetes --user=system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
kubectl config use-context system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
```
```
BOOTSTRAP_TOKEN=$(awk -F "," '{print $1}' /data/apps/kubernetes/conf/token.csv)
kubectl config set-cluster kubernetes --certificate-authority=../ssl/ca.pem --embed-certs=true --server=https://192.168.1.1:6443 --kubeconfig=kubelet-bootstrap.kubeconfig
kubectl config set-credentials kubelet-bootstrap --token=${BOOTSTRAP_TOKEN} --kubeconfig=kubelet-bootstrap.kubeconfig
kubectl config set-context default --cluster=kubernetes --user=kubelet-bootstrap --kubeconfig=kubelet-bootstrap.kubeconfig
kubectl config use-context default --kubeconfig=kubelet-bootstrap.kubeconfig
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap
```
```
kubectl config set-cluster kubernetes --certificate-authority=../ssl/ca.pem --embed-certs=true --server=https://192.168.1.1:6443 --kubeconfig=kube-proxy.kubeconfig
kubectl config set-credentials kube-proxy --client-certificate=../ssl/kube-proxy.pem --client-key=../ssl/kube-proxy-key.pem --embed-certs=true --kubeconfig=kube-proxy.kubeconfig
kubectl config set-context default --cluster=kubernetes --user=kube-proxy --kubeconfig=kube-proxy.kubeconfig
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```
```
/usr/local/bin/kube-controller-manager \
  --secure-port=10257 \
  --bind-address=0.0.0.0 \
  --kubeconfig=/data/apps/kubernetes/conf/kube-controller-manager.kubeconfig \
  --service-cluster-ip-range=10.174.0.0/16 \
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/data/apps/kubernetes/ssl/ca.pem \
  --cluster-signing-key-file=/data/apps/kubernetes/ssl/ca-key.pem \
  --allocate-node-cidrs=true \
  --cluster-cidr=10.175.0.0/16 \
  --root-ca-file=/data/apps/kubernetes/ssl/ca.pem \
  --service-account-private-key-file=/data/apps/kubernetes/ssl/ca-key.pem \
  --leader-elect=true \
  --feature-gates=RotateKubeletServerCertificate=true \
  --controllers=*,bootstrapsigner,tokencleaner \
  --horizontal-pod-autoscaler-sync-period=10s \
  --tls-cert-file=/data/apps/kubernetes/ssl/kube-controller-manager.pem \
  --tls-private-key-file=/data/apps/kubernetes/ssl/kube-controller-manager-key.pem \
  --use-service-account-credentials=true \
  --v=2
```
遇到的问题
calico  ippool  无法更改问题 etcdctl需要配置socket
nacos 连接数据库后隔段时间就无法连接  发现是“Public Key Retrieval is not allowed” allowPublicKeyRetrieval设置为true参数问题
非root用户无法监听1024以下端口问题    setcap cap_net_bind_service=+eip /path/to/application   或者关闭 net.ipv4.ip_unprivileged_port_start
kubernetes集群内部，可以通过coredns rewrite 域名来  重定向域名


