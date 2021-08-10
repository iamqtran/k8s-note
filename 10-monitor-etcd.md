# Monitor ETCD
1. ETCD are external services and all etcd instances communication by using TLS
1. Create a secret configmap which is included
    1. ca.crt
    1. server.crt
    1. server.key
    1. Cli
        ```bash 
        kubectl create secret -n monitoring generic etcd-secrets \
            --from-file=ca.crt \
            --from-file=server.crt \
            --from-file=server.key --dry-run=client -oyaml > etcd_secret.yaml
        ```
1. Update Prometheus CRD using secrets
    ```yaml
    apiVersion: monitoring.coreos.com/v1
    kind: Prometheus
    metadata:
      labels:
        prometheus: k8s
      name: k8s
      namespace: monitoring
    spec:
      alerting:
        alertmanagers:
        - name: alertmanager-main
          namespace: monitoring
          port: web
      image: quay.io/prometheus/prometheus:v2.22.1
      nodeSelector:
        kubernetes.io/os: linux
      podMonitorNamespaceSelector: {}
      podMonitorSelector: {}
      probeNamespaceSelector: {}
      probeSelector: {}
      replicas: 1
      resources:
        requests:
          memory: 400Mi
      secrets:
      - etcd-secrets      
      ruleSelector:
        matchLabels:
          prometheus: k8s
          role: alert-rules
      securityContext:
        fsGroup: 2000
        runAsNonRoot: true
        runAsUser: 1000
      serviceAccountName: prometheus-k8s
      serviceMonitorNamespaceSelector: {}
      serviceMonitorSelector: {}
      version: v2.22.1
    ```
1. Create etcd service object in kube-system namespace
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: kube-etcd
      labels:
        app: kube-etcd
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: https-metrics
          port: 2379
          protocol: TCP
          targetPort: 2379
      selector:
        component: etcd
        tier: control-plane
      type: ClusterIP
    ```
1. Create serviceMonitor for prometheus operator config
    ```yaml
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
      name: kube-etcd
      namespace: monitoring
      labels:
        app: kube-etcd
    spec:
      endpoints:
      - interval: 30s
        port: https-metrics
        scheme: https
        tlsConfig:
          caFile: /etc/prometheus/secrets/etcd-secrets/ca.crt
          certFile: /etc/prometheus/secrets/etcd-secrets/server.crt
          keyFile: /etc/prometheus/secrets/etcd-secrets/server.key
          insecureSkipVerify: true      
      jobLabel: app.kubernetes.io/name
      selector:
        matchLabels:
          app: kube-etcd
      namespaceSelector:
        matchNames:
          - kube-system
    ```
