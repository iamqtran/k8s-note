# Using MinIO
## Create MinIO using Operator
1. Install the MinIO Kubernetes Operator
    1. wget https://github.com/minio/operator/releases/download/v4.1.3/kubectl-minio_4.1.3_linux_amd64 -O kubectl-minio
    1. chmod +x kubectl-minio
    1. mv kubectl-minio /usr/local/bin/
    1. kubectl minio version
1. Initialize the MinIO Kubernetes Operator
    1. kubectl minio init --namespace minio-operator
1. Validate the Operator Installation
    1. kubectl get all --namespace minio-operator
        ```bash
        NAME                                  READY   STATUS    RESTARTS   AGE
        pod/console-59b769c486-cv7zv          1/1     Running   0          81m
        pod/minio-operator-7976b4df5b-rsskl   1/1     Running   0          81m

        NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
        service/console    ClusterIP   10.105.218.94    <none>        9090/TCP,9443/TCP   81m
        service/operator   ClusterIP   10.110.113.146   <none>        4222/TCP,4233/TCP   81m

        NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/console          1/1     1            1           81m
        deployment.apps/minio-operator   1/1     1            1           81m

        NAME                                        DESIRED   CURRENT   READY   AGE
        replicaset.apps/console-59b769c486          1         1         1       81m
        replicaset.apps/minio-operator-7976b4df5b   1         1         1       81m
        ```
## Deploy a MinIO Tenant using the MinIO Plugin
1. Configure the Persistent Volumes
    1. Create a StorageClass for the MinIO local Volumes
        ```yaml
        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
           name: local-storage
        provisioner: kubernetes.io/no-provisioner
        volumeBindingMode: WaitForFirstConsumer
        ```
    1. Create the Required Persistent Volumes - Create 4 Persistent Volumes
        ```yaml
        apiVersion: v1
        kind: PersistentVolume
        metadata:
           name: minio-1
        spec:
           capacity:
              storage: 10G
           accessModes:
           - ReadWriteOnce
           persistentVolumeReclaimPolicy: Delete
           storageClassName: local-storage
           local:
              path: /mnt/minio/ssd1
           nodeAffinity:
              required:
                 nodeSelectorTerms:
                 - matchExpressions:
                    - key: kubernetes.io/os
                      operator: In
                      values:
                      - linux
        ```
1. Create a Namespace for the MinIO Tenant
   1. kubectl create namespace minio-tenant-1
## Using MinIO with HTTPS
1. Create the MinIO Tenant
    ```bash
    kubectl minio tenant create minio-tenant-1 \
            --servers           2              \
            --volumes           4              \
            --capacity          2G             \
            --storage-class     local-storage  \
            --namespace         minio-tenant-1
    ```
    | Argument        | Description                                                                                                                     |
    |-----------------|---------------------------------------------------------------------------------------------------------------------------------|
    | minio-tenant-1  | The name of the MinIO Tenant which the command creates.                                                                         |
    | --servers       | The number of minio servers to deploy across the Kubernetes cluster.                                                            |
    | --volumes       | The number of volumes in the cluster. kubectl minio determines the number of volumes per server by dividing volumes by servers. |
    | --capacity      | The total capacity of the cluster. kubectl minio determines the capacity of each volume by dividing capacity by volumes.        |
    | --storage-class | The Kubernetes StorageClass to use when creating each PVC.                                                                      |
    | --namespace     | The Kubernetes namespace in which to deploy the MinIO Tenant.                                                                   |
1. Configure Access to the Service
    ```bash
    kubectl get svc --namespace minio-tenant-1

    NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
    minio                    ClusterIP   10.98.93.9      <none>        443/TCP             137m
    minio-tenant-1-console   ClusterIP   10.97.87.8      <none>        9090/TCP,9443/TCP   129m
    minio-tenant-1-hl        ClusterIP   None            <none>        9000/TCP            137m
    ```
    1. The **minio** service corresponds to the MinIO Tenant service. Applications should use this service for performing operations against the MinIO Tenant.
    1. The **minio-tenant-1-console** service corresponds to the MinIO Console. Administrators should use this service for accessing the MinIO Console and performing administrative operations on the MinIO Tenant.
    1. The **minio-tenant-1-hl** corresponds to a headless service used to facilitate communication between Pods in the Tenant.
1. Configuration Ingress with HTTPS - <https://github.com/minio/operator/blob/master/docs/nginx-ingress.md>
    1. minio-tenant-1-console Ingress Service
        ```yaml
        apiVersion: extensions/v1beta1                            
        kind: Ingress                                             
        metadata:                                                 
          name: ingress-minio-console
          namespace: minio-tenant-1
          annotations:
            kubernetes.io/ingress.class: "nginx"
            ## Remove if using CA signed certificate
            nginx.ingress.kubernetes.io/proxy-ssl-verify: "off"
            nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
            nginx.ingress.kubernetes.io/rewrite-target: /
            nginx.ingress.kubernetes.io/proxy-body-size: "0"
            nginx.ingress.kubernetes.io/server-snippet: client_max_body_size 0;
            nginx.ingress.kubernetes.io/configuration-snippet: chunked_transfer_encoding off;  
        spec:                                                     
          rules:                                                  
          - host: minconsole.io
            http:
              paths:                                              
              - backend:                                          
                  serviceName: minio-tenant-1-console
                  servicePort: 9443
        ```
    1. minio Ingress Service
        ```yaml
        apiVersion: extensions/v1beta1                            
        kind: Ingress                                             
        metadata:                                                 
          name: ingress-minio-browser
          namespace: minio-tenant-1
          annotations:
            kubernetes.io/ingress.class: "nginx"
            ## Remove if using CA signed certificate
            nginx.ingress.kubernetes.io/proxy-ssl-verify: "off"
            nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
            nginx.ingress.kubernetes.io/rewrite-target: /
            nginx.ingress.kubernetes.io/proxy-body-size: "0"
            nginx.ingress.kubernetes.io/server-snippet: client_max_body_size 0;
            nginx.ingress.kubernetes.io/configuration-snippet: chunked_transfer_encoding off;  
        spec:                                                     
          rules:                                                  
          - host: minbrowser.io
            http:
              paths:
              - backend:
                  serviceName: minio
                  servicePort: 443
        ```
## Access to MinIO Browser - User Access
1. Login to tenant pod
    ```bash
    hadn@k8s-master:~/monitor/minio$ kubectl -n minio-tenant-1 get po
    NAME                             READY   STATUS    RESTARTS   AGE
    tenant-console-85bfff5cc-6z2pn   1/1     Running   0          133m
    tenant-console-85bfff5cc-gqmh2   1/1     Running   0          133m
    tenant-ss-0-0                    1/1     Running   0          133m
    tenant-ss-0-1                    1/1     Running   0          133m
    ```
1. Get Access key
    ```bash
    kubectl -n minio-tenant-1 exec -it tenant-ss-0-0 -- env |grep MINIO_ROOT_USER
    
    MINIO_ROOT_USER=7069081c-d345-448b-b637-92d82cf894a9
    ```
1. Get Secret key
    ```bash
    kubectl -n minio-tenant-1 exec -it tenant-ss-0-0 -- env |grep MINIO_ROOT_PASSWORD

    MINIO_ROOT_PASSWORD=3ebea615-8657-40f5-b718-305cffdad0bc
    ```
## Configuration MinIO Client - mc
1. Create mc folder and config file
    ```bash
    mkdir ~/.mc
    touch ~/.mc/config.json
    ```
1. Content of config.json using minio - <https://github.com/minio/mc/blob/master/docs/minio-client-configuration-files.md>
    ```json
    {
        "version": "10",
        "aliases": {
            "XL": {
                "url": "http://10.98.93.9",
                "accessKey": "7069081c-d345-448b-b637-92d82cf894a9",
                "secretKey": "3ebea615-8657-40f5-b718-305cffdad0bc",
                "api": "S3v4",
                "path": "auto"
            },
            "fs": {
                "url": "http://10.98.93.9",
                "accessKey": "7069081c-d345-448b-b637-92d82cf894a9",
                "secretKey": "3ebea615-8657-40f5-b718-305cffdad0bc",
                "api": "S3v4",
                "path": "auto"
            },
            "play": {
                "url": "http://10.98.93.9",
                "accessKey": "7069081c-d345-448b-b637-92d82cf894a9",
                "secretKey": "3ebea615-8657-40f5-b718-305cffdad0bc",
                "api": "S3v4",
                "path": "auto"
            },
            "s3": {
                "url": "https://10.98.93.9",
                "accessKey": "7069081c-d345-448b-b637-92d82cf894a9",
                "secretKey": "3ebea615-8657-40f5-b718-305cffdad0bc",
                "api": "S3v4",
                "path": "auto"
            }
        }
    }
    ```
1. Create S3 bucket - <https://docs.min.io/docs/minio-client-complete-guide>
    ```bash
    mc mb  s3/alochym --debug --insecure
    ```
1. Upload file to s3/alochym bucket
    ```bash
    mc cp minio.md s3/alochym --insecure --debug 
    ```
1. List content s3/alochym bucket
    ```bash
    mc ls  s3/alochym --debug --insecure
    ```
## Using MinIO without HTTPS
1. Create the MinIO Tenant **--dry-run**
    ```bash
    kubectl minio tenant create minio-tenant-1 \
            --servers           2              \
            --volumes           4           \
            --capacity          2G             \
            --storage-class     local-storage  \
            --namespace         minio-tenant-1 \
            --output |tee alochym.yaml
    ```
1. Disable TLS on MinIO, edit alochym.yaml change **requestAutoCert: true** => **requestAutoCert: false**
1. kubectl apply -f alochym.yaml
1. Access to MinIO Browser - User Access
    ```yaml
    apiVersion: v1
    data:
      accesskey: NzA2OTA4MWMtZDM0NS00NDhiLWI2MzctOTJkODJjZjg5NGE5
      secretkey: M2ViZWE2MTUtODY1Ny00MGY1LWI3MTgtMzA1Y2ZmZGFkMGJj
    kind: Secret
    metadata:
      creationTimestamp: null
      name: tenant-creds-secret
      namespace: tenant
    ```
    1. Get Access key
        ```bash
        echo NzA2OTA4MWMtZDM0NS00NDhiLWI2MzctOTJkODJjZjg5NGE5 | base64 -d
        
        7069081c-d345-448b-b637-92d82cf894a9
        ```
    1. Get Secret key
        ```bash
        echo M2ViZWE2MTUtODY1Ny00MGY1LWI3MTgtMzA1Y2ZmZGFkMGJj | base64 -d

        3ebea615-8657-40f5-b718-305cffdad0bc
        ```
## Reference link
1. <https://mayadata.io/assets/pdf/minio-object-storage-using-openebs-localpv.pdf>
