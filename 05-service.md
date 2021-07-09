# Service

1. What is SERVICE
   - A Kubernetes service is a logical abstraction for a deployed group of pods in a cluster (which all perform the same function)
   - A Service identifies its member Pods using labels selector
   - Reference youtube link <https://www.youtube.com/watch?v=NFApeJRXos4>
1. What is namespace
   1. Kubernetes supports multiple virtual clusters backed by the same physical cluster.
   1. These virtual clusters are called namespaces.
1. What is DNS
   1. The Domain Name System (DNS) is the phonebook of the Internet.
   1. Humans access information online through domain names, like nytimes.com or espn.com.
   1. DNS translates domain names to IP addresses.
1. A syntax record of Service in DNS
   1. `<service_name>.<namespace>.svc.cluster.local`
1. There 5 types of service
   1. ClusterIP -  default type of Service Object
   1. NodePort - Accessing to k8s cluster from External Resource(nodeport range 30000 - 32767)
   1. LoadBalancer - a cloud provider's load balancer
   1. ExternalName - Maps the Service the externalName field (e.g. foo.bar.example.com), by returning a CNAME record
   1. Headless Service
      - Using with StatefulSets Object
      - You can use a headless service when you want a Pod grouping, but don't need a stable IP address
1. Service Object in yaml with ClusterIP
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
1. Create service
   ```bash
    kubectl -n default apply -f nginx-service.yaml
   ```
1. Check ip address of service
   ```bash
    hadn@k8s-master:~$ kubectl get service nginx-service 
    NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
    nginx-service   ClusterIP   10.101.7.54   <none>        80/TCP    87s
   ```
1. Check pods with same as labels selector in service
   ```bash
    hadn@k8s-master:~$ kubectl describe service nginx-service 
    Name:              nginx-service
    Namespace:         default
    Labels:            app=nginx-service
    Annotations:       <none>
    Selector:          app=nginx
    Type:              ClusterIP
    IP:                10.101.7.54
    Port:              nginx-service  80/TCP
    TargetPort:        80/TCP
    Endpoints:         172.16.0.74:80,172.16.0.75:80,172.16.0.77:80
    Session Affinity:  None
    Events:            <none>
   ```
1. Check routing `ip route get pod_ip_address` or `ip link show type veth`
1. Access to nginx pods via service ip address
   ```bash
    hadn@k8s-master:~$ kubectl -n default exec -it busybox -- sh

    [ root@busybox:/ ]$ curl 10.101.7.54
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
   ```
1. Check iptable `sudo iptable-save > alochym.iptables` 
1. Debug `sudo iptable -n -t nat -L KUBE-SERVICES` 
1. Debug network with tshark
   ```bash
    tshark -i eth0 -V -Y "http"
   ```      
1. Deep dive Calico reference <https://www.youtube.com/watch?v=vOo__3GqyxM>
