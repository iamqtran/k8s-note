# Deployment

1. What is Deployment
- Deployment is an easy way to manage a PODS by labels
1. How Deployment works
1. Deployment with Recreate Strategy
   1. Create new ReplicaSet
   1. All pods should be in Ready states
   1. All traffic are routed to new Pods with new software version
   1. All old Pods and old ReplicaSet are deleted
1. Deployment with Rolling Update Strategy - default Strategy
   1. Create new ReplicaSet
   1. Create Pod with new software version in new ReplicaSet
   1. When new Pod ready and then old Pod is deleted
1. Deployment sample
   ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
      name: nginx
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: nginx
        spec:
          containers:
          - image: nginx:1.20-alpine
            name: nginx
   ```
1. Check result
   ```bash
    kubectl get deployment,replicasets,pods

    NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx   3/3     3            3           38m

    NAME                               DESIRED   CURRENT   READY   AGE
    replicaset.apps/nginx-5b7c575f67   3         3         3       38m

    NAME                         READY   STATUS    RESTARTS   AGE
    pod/busybox                  1/1     Running   25         25h
    pod/nginx-5b7c575f67-mpdc9   1/1     Running   0          36m
    pod/nginx-5b7c575f67-mz54w   1/1     Running   0          36m
    pod/nginx-5b7c575f67-xrg5w   1/1     Running   0          33m
   ```

