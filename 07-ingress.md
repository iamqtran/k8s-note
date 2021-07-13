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
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: alochym-rules
    spec:
      rules:
      - http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
    ```
