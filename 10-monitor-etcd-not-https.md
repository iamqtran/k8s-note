# Monitor ETCD
1. ETCD are external services and all etcd instances communication by using TLS
1. ETCD metrics is running on http port
1. Change 
    1. `--listen-metrics-urls=http://127.0.0.1:2381` to `--listen-metrics-urls=http://0.0.0.0:2381`
    1. livenessProbe `host: 0.0.0.0`
    1. startupProbe `host: 0.0.0.0`
1. Create etcd service object in kube-system namespace
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: kube-etcd-no-https
      labels:
        app: kube-etcd-no-https
      namespace: kube-system
    spec:
      clusterIP: None
      ports:
        - name: http-metrics
          port: 2381
          protocol: TCP
          targetPort: 2381
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
       name: kube-etcd-no-http                              
       namespace: monitoring
       labels:
         app: kube-etcd   
    spec:                                                     
       endpoints:                                              
       - interval: 30s                                         
         port: http-metrics                                    
       jobLabel: app.kubernetes.io/name                        
       selector:                                               
         matchLabels:                                          
           app: kube-etcd-no-https
       namespaceSelector:                                      
         matchNames:                                           
         - kube-system 
    ```
1. Reference link
    1. <https://jpweber.io/blog/monitoring-external-etcd-cluster-with-prometheus-operator/>
    1. `curl --cacert ca.crt --cert healthcheck-client.crt --key healthcheck-client.key https://127.0.0.1:2379/metrics -k`
