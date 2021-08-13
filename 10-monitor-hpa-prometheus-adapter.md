# Prometheus Adapter
1. Prometheus Adapter is required for using Custom Metrics API in K8s. It is used primarily for Horizontal Pod Autoscaler to scale based on metrics retrieved from Prometheus. For example, you can create metrics inside of your application and collect them using Prometheus and then you can scale based on those metrics which is really good, because by default K8s is able to scale based only on raw metrics of CPU and memory usage which is not suitable in many cases.
1. using kubectl
    1. kubectl top pod
       1. kubectl top pod -n kube-system |sort -n -k 3
       1. kubectl top pod -n kube-system |sort -n -k 3 -r |head -n 5
    1. kubectl top node
    1. kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/default/pods | python3 -m json.tool
    1. kubectl get --raw /apis/metrics.k8s.io/v1beta1/namespaces/default/pods/<pod_name> | python3 -m json.tool
## Installation
1. kubecctl apply -f prometheus-adapter-serviceAccount.yaml
1. kubecctl apply -f prometheus-adapter-clusterRole.yaml
1. kubecctl apply -f prometheus-adapter-clusterRoleServerResources.yaml
1. kubecctl apply -f prometheus-adapter-clusterRoleAggregatedMetricsReader.yaml
1. kubecctl apply -f prometheus-adapter-clusterRoleBinding.yaml
1. kubecctl apply -f prometheus-adapter-clusterRoleBindingDelegator.yaml
1. kubecctl apply -f prometheus-adapter-roleBindingAuthReader.yaml
1. kubecctl apply -f prometheus-adapter-configMap.yaml
1. kubecctl apply -f prometheus-adapter-apiService.yaml
1. kubecctl apply -f prometheus-adapter-deployment.yaml
1. kubecctl apply -f prometheus-adapter-service.yaml
1. kubecctl apply -f prometheus-adapter-serviceMonitor.yaml
## Horizontal Pod Autoscaler
1. https://dustinspecker.com/posts/scaling-kubernetes-pods-prometheus-metrics/
1. https://cloud.redhat.com/blog/enabling-monitoring-and-scaling-of-your-own-services-application
## KEDA
1. https://www.haproxy.com/blog/autoscaling-with-the-haproxy-kubernetes-ingress-controller-and-keda/
