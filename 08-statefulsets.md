# StatefulSet
1. What is StatefulSet
    1. StatefulSets are replicated groups of Pods. They have certain unique properties
        1. **Each replica POD gets a persistent hostname with a unique index number (database-0, database-1, ...)**
        1. Each replica POD is created in order from hostname lowest number to hostname highest number
        1. When a StatefulSet is deleted, The replica Pods is also deleted in order from hostname highest number to hostname lowest number
1. StaefulSet object should come along with headless service object
    1. What is Headless Service
        - A headless service is a service without a service IP (.spec.clusterIP = NONE).
        - When do DNS lookup on headless service, result will be in multiple records POD IP address
        - A Client will do loadbalance to all backend pods.
    1. Usually using for database or nosql database
        1. **Pod with multiple containers should be using initContainer for setup primary/slave config file in POD**
1. <https://v1-19.docs.kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/>
1. Full Elastic Search example <https://gist.github.com/timfpark/0907199f5e7d8d41c5faa01793cbc54f> 
1. Practice
    1. Create headless service
        ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          creationTimestamp: null
          labels:
            app: nginx-headless-service
          name: nginx-headless-service
        spec:
          ports:
          - name: nginx-headless-service
            port: 80
            protocol: TCP
            targetPort: 80
          selector:
            app: nginx-headless
          clusterIP: None
        ```
    1. Create statefulset with nginx image
        ```yaml
        apiVersion: apps/v1
        kind: StatefulSet
        metadata:
          labels:
            app: nginx-headless
          name: nginx-headless
        spec:
          serviceName: "nginx-headless-service"
          replicas: 2
          selector:
            matchLabels:
              app: nginx-headless
          template:
            metadata:
              labels:
                app: nginx-headless
            spec:
              containers:
              - image: nginx:1.20-alpine
                name: nginx
        ```
    1. Create dnsutils pod for dns lookup
        ```yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: dnsutils
        spec:
          containers:
          - name: dnsutils
            image: gcr.io/kubernetes-e2e-test-images/dnsutils:1.3
            command:
              - sleep
              - "36000"
        ```
    1. Check dns lookup on headless service - syntax **<headless_svc_name>.default.svc.cluster.local**
        ```bash
        hadn@k8s-master:~$ kubectl exec -it dnsutils -- sh
        / # nslookup nginx-headless-service
        Server:		10.96.0.10
        Address:	10.96.0.10#53

        Name:	nginx-headless-service.default.svc.cluster.local
        Address: 172.16.0.105
        Name:	nginx-headless-service.default.svc.cluster.local
        Address: 172.16.0.104
        ```
    1. Check dns lookup on pod hostname - syntax **<pod_name>.<headless_svc_name>.default.svc.cluster.local**
        ```yaml
        hadn@k8s-master:~$ kubectl exec -it dnsutils -- sh
        / # nslookup nginx-headless-1.nginx-headless-service.default.svc.cluster.local
        Server:		10.96.0.10
        Address:	10.96.0.10#53

        Name:	nginx-headless-1.nginx-headless-service.default.svc.cluster.local
        Address: 172.16.0.105

        / # nslookup nginx-headless-0.nginx-headless-service.default.svc.cluster.local
        Server:		10.96.0.10
        Address:	10.96.0.10#53

        Name:	nginx-headless-0.nginx-headless-service.default.svc.cluster.local
        Address: 172.16.0.104
        ```
