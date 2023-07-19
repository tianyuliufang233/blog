---
title: "Kubernetes 自动更新账号"
date: 2025-06-05T12:48:27+08:00
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
traefik标签
```
traefik.ingress.kubernetes.io/whitelist-source-range: "1.2.3.0/24, fe80::/16"  #ingress 白名单
traefik.ingress.kubernetes.io/router.entrypoints: 'websecure, web'  #开启https及http
traefik.ingress.kubernetes.io/router.middlewares: default-https-redirect-scheme@kubernetescrd   #http重定向到https
```
创建cd-user：
```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployments-update
rules:
- apiGroups:
  - "apps"
  resources:
  - deployments
  verbs:
  - get
  - list
  - watch
  - update
  - rollout
  - patch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: datacenter-dev-cd-user-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: deployments-update
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: datacenter-dev-cd-user
```
datacenter-dev-cd-user.json
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
# 命令
```
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes datacenter-dev-cd-user.json | cfssljson  -bare datacenter-dev-cd-user #生成ssl
#生成kubeconfig
kubectl config set-cluster kubernetes --certificate-authority=ca.pem --embed-certs=true --server=https://172.16.1.94:6443 --kubeconfig=datacenter-dev-cd-user.config
kubectl config set-credentials datacenter-dev-cd-user --client-certificate=datacenter-dev-cd-user.pem --client-key=datacenter-dev-cd-user-key.pem --embed-certs=true --kubeconfig=datacenter-dev-cd-user.config
kubectl config set-context kubernetes --cluster=kubernetes --user=datacenter-dev-cd-user --kubeconfig=datacenter-dev-cd-user.config
kubectl config use-context kubernetes --kubeconfig=datacenter-dev-cd-user.config
```

