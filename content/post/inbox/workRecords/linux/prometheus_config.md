---
title: "Prometheus_config"
date: 2023-10-24T09:43:29+08:00
categories:
- 收件箱
- linxu相关
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
```
# A scrape configuration for running Prometheus on a Kubernetes cluster.
# This uses separate scrape configs for cluster components (i.e. API server, node)
# and services to allow each to use different authentication configs.
#
# Kubernetes labels will be added as Prometheus labels on metrics via the
# `labelmap` relabeling action.
#
# If you are using Kubernetes 1.7.2 or earlier, please take note of the comments
# for the kubernetes-cadvisor job; you will need to edit or remove this job.

# Scrape config for API servers.
#
# Kubernetes exposes API servers as endpoints to the default/kubernetes
# service so this uses `endpoints` role and uses relabelling to only keep
# the endpoints associated with the default/kubernetes service using the
# default named port `https`. This works for single API server deployments as
# well as HA API server deployments.
scrape_configs:
  - job_name: "kubernetes-apiservers"

    kubernetes_sd_configs:
      - role: endpoints

    # Default to scraping over https. If required, just disable this or change to
    # `http`.
    scheme: https

    # This TLS & authorization config is used to connect to the actual scrape
    # endpoints for cluster components. This is separate to discovery auth
    # configuration because discovery & scraping are two separate concerns in
    # Prometheus. The discovery auth config is automatic if Prometheus runs inside
    # the cluster. Otherwise, more config options have to be provided within the
    # <kubernetes_sd_config>.
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      # If your node certificates are self-signed or use a different CA to the
      # master CA, then disable certificate verification below. Note that
      # certificate verification is an integral part of a secure infrastructure
      # so this should only be disabled in a controlled environment. You can
      # disable certificate verification by uncommenting the line below.
      #
      # insecure_skip_verify: true
    authorization:
      credentials_file: /var/run/secrets/kubernetes.io/serviceaccount/token

    # Keep only the default/kubernetes service endpoints for the https port. This
    # will add targets for each API server which Kubernetes adds an endpoint to
    # the default/kubernetes service.
    relabel_configs:
      - source_labels:
          [
            __meta_kubernetes_namespace,
            __meta_kubernetes_service_name,
            __meta_kubernetes_endpoint_port_name,
          ]
        action: keep
        regex: default;kubernetes;https

  # Scrape config for nodes (kubelet).
  #
  # Rather than connecting directly to the node, the scrape is proxied though the
  # Kubernetes apiserver.  This means it will work if Prometheus is running out of
  # cluster, or can't connect to nodes for some other reason (e.g. because of
  # firewalling).
  - job_name: "kubernetes-nodes"

    # Default to scraping over https. If required, just disable this or change to
    # `http`.
    scheme: https

    # This TLS & authorization config is used to connect to the actual scrape
    # endpoints for cluster components. This is separate to discovery auth
    # configuration because discovery & scraping are two separate concerns in
    # Prometheus. The discovery auth config is automatic if Prometheus runs inside
    # the cluster. Otherwise, more config options have to be provided within the
    # <kubernetes_sd_config>.
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      # If your node certificates are self-signed or use a different CA to the
      # master CA, then disable certificate verification below. Note that
      # certificate verification is an integral part of a secure infrastructure
      # so this should only be disabled in a controlled environment. You can
      # disable certificate verification by uncommenting the line below.
      #
      # insecure_skip_verify: true
    authorization:
      credentials_file: /var/run/secrets/kubernetes.io/serviceaccount/token

    kubernetes_sd_configs:
      - role: node

    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)

  # Scrape config for Kubelet cAdvisor.
  #
  # This is required for Kubernetes 1.7.3 and later, where cAdvisor metrics
  # (those whose names begin with 'container_') have been removed from the
  # Kubelet metrics endpoint.  This job scrapes the cAdvisor endpoint to
  # retrieve those metrics.
  #
  # In Kubernetes 1.7.0-1.7.2, these metrics are only exposed on the cAdvisor
  # HTTP endpoint; use the "/metrics" endpoint on the 4194 port of nodes. In
  # that case (and ensure cAdvisor's HTTP server hasn't been disabled with the
  # --cadvisor-port=0 Kubelet flag).
  #
  # This job is not necessary and should be removed in Kubernetes 1.6 and
  # earlier versions, or it will cause the metrics to be scraped twice.
  - job_name: "kubernetes-cadvisor"

    # Default to scraping over https. If required, just disable this or change to
    # `http`.
    scheme: https

    # Starting Kubernetes 1.7.3 the cAdvisor metrics are under /metrics/cadvisor.
    # Kubernetes CIS Benchmark recommends against enabling the insecure HTTP
    # servers of Kubernetes, therefore the cAdvisor metrics on the secure handler
    # are used.
    metrics_path: /metrics/cadvisor

    # This TLS & authorization config is used to connect to the actual scrape
    # endpoints for cluster components. This is separate to discovery auth
    # configuration because discovery & scraping are two separate concerns in
    # Prometheus. The discovery auth config is automatic if Prometheus runs inside
    # the cluster. Otherwise, more config options have to be provided within the
    # <kubernetes_sd_config>.
    tls_config:
      ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      # If your node certificates are self-signed or use a different CA to the
      # master CA, then disable certificate verification below. Note that
      # certificate verification is an integral part of a secure infrastructure
      # so this should only be disabled in a controlled environment. You can
      # disable certificate verification by uncommenting the line below.
      #
      # insecure_skip_verify: true
    authorization:
      credentials_file: /var/run/secrets/kubernetes.io/serviceaccount/token

    kubernetes_sd_configs:
      - role: node

    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)

  # Example scrape config for service endpoints.
  #
  # The relabeling allows the actual service scrape endpoint to be configured
  # for all or only some endpoints.
  - job_name: "kubernetes-service-endpoints"

    kubernetes_sd_configs:
      - role: endpoints

    relabel_configs:
      # Example relabel to scrape only endpoints that have
      # "example.io/should_be_scraped = true" annotation.
      #  - source_labels: [__meta_kubernetes_service_annotation_example_io_should_be_scraped]
      #    action: keep
      #    regex: true
      #
      # Example relabel to customize metric path based on endpoints
      # "example.io/metric_path = <metric path>" annotation.
      #  - source_labels: [__meta_kubernetes_service_annotation_example_io_metric_path]
      #    action: replace
      #    target_label: __metrics_path__
      #    regex: (.+)
      #
      # Example relabel to scrape only single, desired port for the service based
      # on endpoints "example.io/scrape_port = <port>" annotation.
      #  - source_labels: [__address__, __meta_kubernetes_service_annotation_example_io_scrape_port]
      #    action: replace
      #    regex: ([^:]+)(?::\d+)?;(\d+)
      #    replacement: $1:$2
      #    target_label: __address__
      #
      # Example relabel to configure scrape scheme for all service scrape targets
      # based on endpoints "example.io/scrape_scheme = <scheme>" annotation.
      #  - source_labels: [__meta_kubernetes_service_annotation_example_io_scrape_scheme]
      #    action: replace
      #    target_label: __scheme__
      #    regex: (https?)
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: service

  # Example scrape config for probing services via the Blackbox Exporter.
  #
  # The relabeling allows the actual service scrape endpoint to be configured
  # for all or only some services.
  - job_name: "kubernetes-services"

    metrics_path: /probe
    params:
      module: [http_2xx]

    kubernetes_sd_configs:
      - role: service

    relabel_configs:
      # Example relabel to probe only some services that have "example.io/should_be_probed = true" annotation
      #  - source_labels: [__meta_kubernetes_service_annotation_example_io_should_be_probed]
      #    action: keep
      #    regex: true
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: service

  # Example scrape config for probing ingresses via the Blackbox Exporter.
  #
  # The relabeling allows the actual ingress scrape endpoint to be configured
  # for all or only some services.
  - job_name: "kubernetes-ingresses"

    metrics_path: /probe
    params:
      module: [http_2xx]

    kubernetes_sd_configs:
      - role: ingress

    relabel_configs:
      # Example relabel to probe only some ingresses that have "example.io/should_be_probed = true" annotation
      #  - source_labels: [__meta_kubernetes_ingress_annotation_example_io_should_be_probed]
      #    action: keep
      #    regex: true
      - source_labels:
          [
            __meta_kubernetes_ingress_scheme,
            __address__,
            __meta_kubernetes_ingress_path,
          ]
        regex: (.+);(.+);(.+)
        replacement: ${1}://${2}${3}
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_ingress_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
      - source_labels: [__meta_kubernetes_ingress_name]
        target_label: ingress

  # Example scrape config for pods
  #
  # The relabeling allows the actual pod scrape to be configured
  # for all the declared ports (or port-free target if none is declared)
  # or only some ports.
  - job_name: "kubernetes-pods"

    kubernetes_sd_configs:
      - role: pod

    relabel_configs:
      # Example relabel to scrape only pods that have
      # "example.io/should_be_scraped = true" annotation.
      #  - source_labels: [__meta_kubernetes_pod_annotation_example_io_should_be_scraped]
      #    action: keep
      #    regex: true
      #
      # Example relabel to customize metric path based on pod
      # "example.io/metric_path = <metric path>" annotation.
      #  - source_labels: [__meta_kubernetes_pod_annotation_example_io_metric_path]
      #    action: replace
      #    target_label: __metrics_path__
      #    regex: (.+)
      #
      # Example relabel to scrape only single, desired port for the pod
      # based on pod "example.io/scrape_port = <port>" annotation.
      #  - source_labels: [__address__, __meta_kubernetes_pod_annotation_example_io_scrape_port]
      #    action: replace
      #    regex: ([^:]+)(?::\d+)?;(\d+)
      #    replacement: $1:$2
      #    target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: pod
```