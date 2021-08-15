# Monitor kube-state-metrics
1. kube-state-metrics is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects. It is focused on the health of the various objects.
    1. Deployments.
    1. Pods.
    1. Services.
    1. StatefulSets.
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
    kubectl auth can-i --as system:serviceaccount:monitoring:prometheus-k8s -n monitoring get/list/watch pods/services/endpoints
    ```
1. kubectl apply -f kube-state-metrics-serviceAccount.yaml
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      labels:
        app.kubernetes.io/name: kube-state-metrics
        app.kubernetes.io/version: v1.9.7
      name: kube-state-metrics
      namespace: monitoring
    ```
1. kubectl apply -f kube-state-metrics-clusterRole.yaml 
1. kubectl apply -f kube-state-metrics-clusterRoleBinding.yaml 
1. kubectl apply -f kube-state-metrics-deployment.yaml
1. kubectl apply -f kube-state-metrics-service.yaml
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app.kubernetes.io/name: kube-state-metrics
        app.kubernetes.io/version: v1.9.7
      name: kube-state-metrics
      namespace: monitoring
    spec:
      clusterIP: None
      ports:
      - name: https-main
        port: 8443
        targetPort: https-main
      - name: https-self
        port: 9443
        targetPort: https-self
      selector:
        app.kubernetes.io/name: kube-state-metrics
    ```
1. kubectl apply -f kube-state-metrics-serviceMonitor.yaml 
    ```yaml
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
      labels:
        app.kubernetes.io/name: kube-state-metrics
        app.kubernetes.io/version: 1.9.7
      name: kube-state-metrics
      namespace: monitoring
    spec:
      endpoints:
      - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
        honorLabels: true
        interval: 30s
        port: https-main
        relabelings:
        - action: labeldrop
          regex: (pod|service|endpoint|namespace)
        scheme: https
        scrapeTimeout: 30s
        tlsConfig:
          insecureSkipVerify: true
      - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
        interval: 30s
        port: https-self
        scheme: https
        tlsConfig:
          insecureSkipVerify: true
      jobLabel: app.kubernetes.io/name
      selector:
        matchLabels:
          app.kubernetes.io/name: kube-state-metrics
    ```
