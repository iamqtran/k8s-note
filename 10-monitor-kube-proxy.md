# Montior kube-proxy
1. Create kube-proxy service in kube-system namespace
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: kube-proxy
      labels:
        app: kube-proxy
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: http-metrics
          port: 10249
          protocol: TCP
          targetPort: 10249
      selector:
        k8s-app: kube-proxy
      type: ClusterIP
    ```
1. Check role prometheus-k8s on kube-system
    ```bash
    kubectl -n kube-system describe role prometheus-k8s
    ```
1. Check rolebinding prometheus-k8s on kube-system
    ```bash
    kubectl -n kube-system describe rolebindings prometheus-k8s
    ```
1. Check "prometheus-k8s" has permission on kube-system namespace
    ```bash
    kubectl auth can-i --as system:serviceaccount:monitoring:prometheus-k8s -n kube-system get/list/watch pods/services/endpoints
    ```
1. Change kube-proxy configmap metricsBindAddress: "" => metricsBindAddress: "0.0.0.0:10249" in kube-system namespace.
    ```yaml
    apiVersion: v1
    data:
      config.conf: |-
        apiVersion: kubeproxy.config.k8s.io/v1alpha1
        bindAddress: 0.0.0.0
        bindAddressHardFail: false
        clientConnection:
          acceptContentTypes: ""
          burst: 0
          contentType: ""
          kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
          qps: 0
        clusterCIDR: 172.16.0.0/16
        configSyncPeriod: 0s
        conntrack:
          maxPerCore: null
          min: null
          tcpCloseWaitTimeout: null
          tcpEstablishedTimeout: null
        detectLocalMode: ""
        enableProfiling: false
        healthzBindAddress: ""
        hostnameOverride: ""
        iptables:
          masqueradeAll: false
          masqueradeBit: null
          minSyncPeriod: 0s
          syncPeriod: 0s
        ipvs:
          excludeCIDRs: null
          minSyncPeriod: 0s
          scheduler: ""
          strictARP: false
          syncPeriod: 0s
          tcpFinTimeout: 0s
          tcpTimeout: 0s
          udpTimeout: 0s
        kind: KubeProxyConfiguration
        metricsBindAddress: "0.0.0.0:10249"
      kubeconfig.conf: |-
        apiVersion: v1
        kind: Config
        clusters:
        - cluster:
            certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
            server: https://k8s-master:6443
          name: default
        contexts:
        - context:
            cluster: default
            namespace: default
            user: default
          name: default
        current-context: default
        users:
        - name: default
          user:
            tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    kind: ConfigMap
    metadata:
      labels:
        app: kube-proxy
      name: kube-proxy
      namespace: kube-system
    ```
1. Delete all kube-proxy pod in kube-system namespace to reload configmap
    ```bash
    kubectl -n kube-system delete po kube-proxy-f9k2n
    ```
1. Create serviceMonitor for prometheus operator
    ```yaml
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
      name: kube-proxy
      namespace: monitoring
      labels:
        app: kube-proxy
    spec:
      jobLabel: jobLabel
      selector:
        matchLabels:
          app: kube-proxy
      namespaceSelector:
        matchNames:
          - kube-system
      endpoints:
      - port: http-metrics
    ```
