# What is POD

1. Pods are the smallest deployable units of computing that you can create and manage in Kubernetes
1. A Pod (as in a pod of whales or pea pod) is a group of one or more containers with shared storage/network resources
1. There are two types of Pods
   1. Single-container Pods
      1. <https://drek4537l1klr.cloudfront.net/luksa/Figures/03fig01_alt.jpg>
   1. Multi-container Pods
      1. <https://drek4537l1klr.cloudfront.net/luksa/Figures/03fig01_alt.jpg>
1. A Pod networking
   1. Have ip address and ports to communicate to other Pods
      1. <https://drek4537l1klr.cloudfront.net/luksa/Figures/03fig01_alt.jpg>
   1. The IP address and ports, which are shared among the containers running in a Pod if there is more than one container
      1. Containers running in a Pod to communicate to other containers by loopback interface - 127.0.0.1
      1. <https://learnk8s.io/sidecar-containers-patterns>
1. A Pod storage. There are 2 type of storages in kubernetes
   1. Persistence Storage
   1. Non-persistent Storage
1. Pods are managed by label
1. Pods lifecycle
   1. Pending
   1. Running
   1. Succeeded
   1. Failed
   1. Unknown
   1. Creating
   1. CrashLoopBackOff
1. Pods also contain storage resources for their containers:
   1. Storage: Pods can specify a set of shared storage volumes that can be shared among the containers.
   ![title](./kubernetes-how-it-works-diagrams-03.jpg)
1. Deploy a POD to kubernetes cluster
   1. Create nginx.yaml
      ```yaml
        // kubectl run nginx --image=nginx:1.18-alpine --port=80 --labels='env=production,app=nginx' --dry-run=client -oyaml > multi-container-pod.yaml

        apiVersion: v1
        kind: Pod
        metadata:
          creationTimestamp: null
          labels:
            app: nginx
            env: production
          name: nginx
        spec:
          containers:
          - image: nginx:1.18-alpine
            name: nginx
            ports:
            - containerPort: 80
      ```
   1. Using kubectl create pod
      ```bash
      kubectl -n default apply -f nginx.yaml
      ```
   1. Using kubectl check pod which is running or not
      ```bash
      kubectl -n default get pods
      ```
1. Check nginx web server
   1. Create busybox in same namespace of nginx pod
      ```yaml
      apiVersion: v1                                        
      kind: Pod                                             
      metadata:                                             
          name: busybox                                         
      spec:                                                 
          containers:                                         
          - image: radial/busyboxplus:curl
            name: busybox                                 
            command:
              - sh
              - -c
              - sleep 3600    
      ```
   1. Login into busybox and curl to nginx pod's IP address
      ```bash
      kubectl -n default exec -it -- sh

      curl http://nginx_pods_ip_address
      ```
