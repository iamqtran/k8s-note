# Ingress
1. Ingress Controller exposes HTTP and HTTPS routes from outside the cluster to services within the cluster.
1. Traffic routing is controlled by rules defined on the Ingress resource(route/rule).
1. Exposing services other than HTTP and HTTPS to the internet typically uses a service of type:
    1. Service.Type=NodePort(Bare Server)
    1. Service.Type=LoadBalancer(cloud provider)
1. There are 2 type of ingress
    1. Ingress resources - A group of rules for routing
    1. Ingress controller
        1. An ingress controller is responsible for reading the Ingress Resource information and processing that data accordingly
        1. Ingress controllers are pods. They’re built using reverse proxies that have been active in the market for years.
        1. You need to expose them to the outside via a Service with a type of either NodePort or LoadBalancer.
1. Install nginx-ingess controller
1. Create service with type=NodePort for nginx-ingress pod
1. All setup will be using <https://github.com/kubernetes/ingress-nginx/blob/master/deploy/static/provider/baremetal/deploy.yaml>
1. Create Ingress-rules
    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: grafana
      namespace: default
    spec:
      rules:
      - host: www.nginx-ingress.com
        http:
          paths:
          - backend:
              serviceName: nginx-service
              servicePort: 80
    ```
1. Create Nginx service in **default namespace**
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-service
      name: nginx-service
    spec:
      ports:
      - name: nginx-service
        port: 80
        protocol: TCP
        targetPort: 80
      selector:
        app: nginx
      type: ClusterIP
    status:
      loadBalancer: {}
    ```
