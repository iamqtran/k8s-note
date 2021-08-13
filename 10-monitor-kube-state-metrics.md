# Monitor kube-state-metrics
1. kube-state-metrics is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects. (See examples in the Metrics section below.) It is not focused on the health of the individual Kubernetes components, but rather on the health of the various objects inside, such as deployments, nodes and pods.
1. kubectl apply -f kube-state-metrics-serviceAccount.yaml
1. kubectl apply -f kube-state-metrics-clusterRole.yaml 
1. kubectl apply -f kube-state-metrics-clusterRoleBinding.yaml 
1. kubectl apply -f kube-state-metrics-deployment.yaml
1. kubectl apply -f kube-state-metrics-service.yaml
1. kubectl apply -f kube-state-metrics-serviceMonitor.yaml 

