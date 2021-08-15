# Monitor Kube Scheduler
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
1. Create kube-scheduler service in kube-system namespace
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: kube-scheduler
      labels:
        app.kubernetes.io/name: kube-scheduler
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: https-metrics
          port: 10259
          protocol: TCP
          targetPort: 10259
      selector:
        component: kube-scheduler
      type: ClusterIP
    ```
1. Create kube-scheduler serviceMonitor
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: kube-scheduler
      labels:
        app.kubernetes.io/name: kube-scheduler
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: https-metrics
          port: 10259
          protocol: TCP
          targetPort: 10259
      selector:
        component: kube-scheduler
      type: ClusterIP
    ```
1. Edit `/etc/kubernetes/manifests/kube-scheduler.yaml` to change 127.0.0.1 => 0.0.0.0
    ```bash
    sed -i 's/127.0.0.1/0.0.0.0/g' /etc/kubernetes/manifests/kube-scheduler.yaml
    ```
1. kube-scheduler file
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      creationTimestamp: null
      labels:
        component: kube-scheduler
        tier: control-plane
      name: kube-scheduler
      namespace: kube-system
    spec:
      containers:
      - command:
        - kube-scheduler
        - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
        - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
        - --bind-address=0.0.0.0
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --leader-elect=true
        - --port=0
        image: k8s.gcr.io/kube-scheduler:v1.19.0
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 8
          httpGet:
            host: 0.0.0.0
            path: /healthz
            port: 10259
            scheme: HTTPS
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 15
        name: kube-scheduler
        resources:
          requests:
            cpu: 100m
        startupProbe:
          failureThreshold: 24
          httpGet:
            host: 0.0.0.0
            path: /healthz
            port: 10259
            scheme: HTTPS
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 15
        volumeMounts:
        - mountPath: /etc/kubernetes/scheduler.conf
          name: kubeconfig
          readOnly: true
      hostNetwork: true
      priorityClassName: system-node-critical
      volumes:
      - hostPath:
          path: /etc/kubernetes/scheduler.conf
          type: FileOrCreate
        name: kubeconfig
    status: {}
    ```
