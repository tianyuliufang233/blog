---
title: "Kubernetes Crane Scheduler作为第二调度器"
date: 2025-06-05T12:42:56+08:00
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

# 背景： 
kubernetes scheduler 使用resource 的request和limit进行打分。当没配置request和limit的时候调度就会失衡。
crane-scheduler获取prometheus监控值对node进行打分，更贴近真实资源占用。

# 步骤：
1. 安装prometheus，安装node-exporter 获取指标
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
```

2. 配置prometheus kubernetes-sd  自发现

prometheus.yml
```
- job_name: "prometheus-node-export"
  kubernetes_sd_configs:
    - role: endpoints
      namespaces:
        names:
        - crane-system
      api_server: 'https://172.27.0.15:6443/'
      bearer_token_file: /opt/bitnami/prometheus/conf/rules/token
      tls_config:
        insecure_skip_verify: true
  scheme: http
  tls_config:
    ca_file: /opt/bitnami/prometheus/conf/rules/ca.pem
  authorization:
    credentials_file: /opt/bitnami/prometheus/conf/rules/token
```
rule_files 目录中加入schedule_rule.yml  并加入读取权限
```
groups:
- name: cpu_mem_usage_active
  interval: 30s
  rules:
  - record: cpu_usage_active
    expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[30s])) * 100)
  - record: mem_usage_active
    expr: 100*(1-node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes)
- name: cpu-usage-5m
  interval: 5m
  rules:
  - record: cpu_usage_max_avg_1h
    expr: max_over_time(cpu_usage_avg_5m[1h])
  - record: cpu_usage_max_avg_1d
    expr: max_over_time(cpu_usage_avg_5m[1d])
- name: cpu-usage-1m
  interval: 1m
  rules:
  - record: cpu_usage_avg_5m
    expr: avg_over_time(cpu_usage_active[5m])
- name: mem-usage-5m
  interval: 5m
  rules:
  - record: mem_usage_max_avg_1h
    expr: max_over_time(mem_usage_avg_5m[1h])
  - record: mem_usage_max_avg_1d
    expr: max_over_time(mem_usage_avg_5m[1d])
- name: mem-usage-1m
  interval: 1m
  rules:
  - record: mem_usage_avg_5m
    expr: avg_over_time(mem_usage_active[5m])
```
3. helm安装crane-scheduler作为第二scheduler
```
helm repo add crane https://gocrane.github.io/helm-charts
helm install scheduler -n crane-system --create-namespace --set global.prometheusAddr="REPLACE_ME_WITH_PROMETHEUS_ADDR" crane/scheduler
```
4. 测试
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpu-stress
spec:
  selector:
    matchLabels:
      app: cpu-stress
  replicas: 1
  template:
    metadata:
      labels:
        app: cpu-stress
    spec:
      schedulerName: crane-scheduler
      hostNetwork: true
      tolerations:
      - key: node.kubernetes.io/network-unavailable
        operator: Exists
        effect: NoSchedule
      containers:
      - name: stress
        image: docker.io/gocrane/stress:latest
        command: ["stress", "-c", "1"]
        resources:
          requests:
            memory: "1Gi"
            cpu: "1"
          limits:
            memory: "1Gi"
            cpu: "1"
```

# 测试结果
```
Type    Reason     Age   From             Message
----    ------     ----  ----             -------
Normal  Scheduled  28s   crane-scheduler  Successfully assigned default/cpu-stress-7669499b57-zmrgb to vm-162-247-ubuntu
```





# 参考：
[GitHub - gocrane/crane-scheduler：Crane 调度器是一个 Kubernetes 调度器，可以根据实际节点负载调度 Pod。](https://github.com/gocrane/crane-scheduler)