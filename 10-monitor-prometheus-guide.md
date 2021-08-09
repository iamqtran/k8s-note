# Monitor Kubernetes with Prometheus
## Prometheus
## Prometheus Operator
1. (**CustomResourceDefinitions**)[https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/]
1. **Prometheus**, which defines a desired Prometheus deployment.
1. **Alertmanager**, which defines a desired Alertmanager deployment.
1. **ThanosRuler**, which defines a desired Thanos Ruler deployment.
1. **ServiceMonitor**, which declaratively specifies how groups of Kubernetes services should be monitored. The Operator automatically generates Prometheus scrape configuration based on the current state of the objects in the API server.
1. **PodMonitor**, which declaratively specifies how group of pods should be monitored. The Operator automatically generates Prometheus scrape configuration based on the current state of the objects in the API server.
1. **Probe**, which declaratively specifies how groups of ingresses or static targets should be monitored. The Operator automatically generates Prometheus scrape configuration based on the definition.
1. **PrometheusRule**, which defines a desired set of Prometheus alerting and/or recording rules. The Operator generates a rule file, which can be used by Prometheus instances.
1. **AlertmanagerConfig**, which declaratively specifies subsections of the Alertmanager configuration, allowing routing of alerts to custom receivers, and setting inhibit rules.
## TroubleShooting
1. Get Token from prometheus pod
    ```bash
    1. kubectl exec -it -n monitoring prometheus-k8s-0 -c prometheus -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
    2. TOKEN=step1
    3. curl -v -s -k -H "Authorization: Bearer $Token" https://10.0.0.10:9100/metrics
    ```
## Grafana
