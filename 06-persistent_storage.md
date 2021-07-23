# Persistent Storage
1. What Volume
   1. All data from the container of PODS will be deleted when the container of POD restarts
   1. Volume is created to solve that problem. Data save in volume which will be persist across container restart
   1. There are 2 type of volume:
      1. Static provision volume
      1. Dynamic provision volume - Storage class
   1. Static provision volume:
      1. HostPath: The volume itself does not contain scheduling information. If you want to fix each pod on a node, you need to configure scheduling information, such as nodeSelector, for the pod.
      1. LocalVolume (**Local Persistence Volume Static Provisioner**): The volume itself contains scheduling information, and the pods using this volume will be fixed on a specific node, which can ensure data continuity. 
      1. emptyDir:
         - Containers in the Pod can all read and write the same files in the emptyDir volume
         - When a Pod is removed from a node for any reason, the data in the emptyDir is deleted forever
1. Persistent Volume
    1. is a piece of storage in the cluster that captures the implementation details of the storage. A PV is provisioned by a cluster administrator.
    1. AccessMode:
        1. ReadWriteOnce – the volume can be mounted as read-write by a single node ... ***RWO***
        1. ReadOnlyMany – the volume can be mounted read-only by many nodes ... ***ROX***
        1. ReadWriteMany – the volume can be mounted as read-write by many nodes ... ***RWX***
    1. Status
        1. Available - ***A free resource not yet bound to a claim.***
        1. Bound - ***The volume is bound to a claim.***
        1. Released - ***The claim was deleted, but the resource is not yet reclaimed by the cluster.***
        1. Failed - ***The volume has failed its automatic reclamation.***
1. Persistent Volume Claim
    1. is a request for storage.
    1. PersistentVolumeClaim objects request a specific size, access mode, and StorageClass for the PersistentVolume.
    1. If a PersistentVolume that satisfies the request exists or can be provisioned, the PersistentVolumeClaim is bound to that PersistentVolume.
1. Storage Class is support dynamic provision volume that means no need to create Persistent Volume before using
1. Practice example
    1. EmptyDir should be using with initContainer for configuration files of main container
        1. Pod with Persistent Volume Claim
            ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: nginx
            spec:
              containers:
              - name: nginx
                image: nginx:1.20-alpine
                volumeMounts:
                - name: nginx-storage
                  mountPath: /data/nginx
              volumes:
              - name: nginx-storage
                emptyDir: {}
            ```
    1. HostPath
        1. Pod with hostpath as readonly
            ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: pod-node-exporter
              labels:
                app: node-exporter
            spec:
                  containers:
                  - args:
                    - --web.listen-address=:9100
                    - --path.sysfs=/host/sys
                    - --path.rootfs=/host/root
                    - --no-collector.wifi
                    - --no-collector.hwmon
                    - --collector.filesystem.ignored-mount-points=^/(dev|proc|sys|var/lib/docker/.+|var/lib/kubelet/pods/.+)($|/)
                    - --collector.netclass.ignored-devices=^(veth.*|[a-z0-9]+@if\d+)$
                    - --collector.netdev.device-exclude=^(veth.*|[a-z0-9]+@if\d+)$
                    image: quay.io/prometheus/node-exporter:v1.1.2
                    name: node-exporter
                    volumeMounts:
                    - mountPath: /host/sys
                      mountPropagation: HostToContainer #Bidirectional
                      name: sys
                      readOnly: true
                    - mountPath: /host/root
                      mountPropagation: HostToContainer
                      name: root
                      readOnly: true
                  securityContext:
                    runAsNonRoot: true
                    runAsUser: 65534
                  volumes:
                  - hostPath:
                      path: /sys
                    name: sys
                  - hostPath:
                      path: /
                    name: root
            ```
        1. Pod with hostpath read and write
            ```yaml
            apiVersion: v1                                        
            kind: Pod                                             
            metadata:                                             
              creationTimestamp: null                             
              labels:                                             
                app: nginx                                        
                env: production                                   
              name: nginx-hostpath
            spec:                                                 
              containers:                                         
              - image: nginx:1.20-alpine                          
                name: nginx                                 
                ports:                                            
                - containerPort: 80 
                securityContext:
                  privileged: true
                  capabilities:
                    add: ["SYS_ADMIN"]
                  allowPrivilegeEscalation: true
                volumeMounts:
                  - name: dynamic-volume
                    mountPropagation: "Bidirectional"
                    mountPath: "/dynamic-volume"
              volumes:
                - name: dynamic-volume
                  hostPath:
                    path: /mnt/dynamic-volume
                    type: DirectoryOrCreate      
            ```
        1. Persistent volume with hostpath
            ```yaml
            apiVersion: v1
            kind: PersistentVolume
            metadata:
              name: pv-volume
              labels:
                type: local
            spec:
              capacity:
                storage: 10Gi
              accessModes:
                - ReadWriteOnce
              hostPath:
                path: "/mnt/data"
            ```
        1. Using Persistent Volume Claim
            ```yaml
            apiVersion: v1
            kind: PersistentVolumeClaim
            metadata:
              name: pv-claim
            spec:
              accessModes:
                - ReadWriteOnce
              resources:
                requests:
                  storage: 3Gi
            ```
        1. Pod using Persistent Volume Claim
            ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: pv-pod
            spec:
              volumes:
                - name: task-pv-storage
                  persistentVolumeClaim:
                    claimName: pv-claim
              containers:
                - name: task-pv-container
                  image: nginx:1.20-alpine
                  ports:
                    - containerPort: 80
                  volumeMounts:
                    - mountPath: "/usr/share/nginx/html"
                      name: task-pv-storage
            ```
    1. LocalVolume
        1. Create mkdir -p /mnt/disk/ssd
        1. Storage Class
            ```yaml
            kind: StorageClass
            apiVersion: storage.k8s.io/v1
            metadata:
              name: local-storage
            provisioner: kubernetes.io/no-provisioner
            volumeBindingMode: WaitForFirstConsumer
            ```
        1. Persistent Volume
            ```yaml
            apiVersion: v1
            kind: PersistentVolume
            metadata:
              name: local-storage
            spec:
              capacity:
                storage: 10Gi
              accessModes:
              - ReadWriteOnce
              persistentVolumeReclaimPolicy: Retain
              storageClassName: local-storage
              local:
                path: /mnt/disk/ssd
              nodeAffinity:
                required:
                  nodeSelectorTerms:
                  - matchExpressions:
                    - key: kubernetes.io/hostname
                      operator: In
                      values:
                      - worker-10-0-0-50
            ```
        1. Persistent Volume Claim
            ```yaml
            kind: PersistentVolumeClaim
            apiVersion: v1
            metadata:
              name: local-storage
            spec:
              accessModes:
              - ReadWriteOnce
              storageClassName: local-storage
              resources:
                requests:
                  storage: 10Gi
            ```
        1. Pod with Persistent Volume Claim
            ```yaml
            apiVersion: v1
            kind: Pod
            metadata:
              name: busybox-local
              labels:
                name: busybox-local
            spec:
              containers:
              - name: app
                image: busybox
                command: ['sh', '-c', 'echo "The local volume is mounted!" > /mnt/test.txt && sleep 3600']
                volumeMounts:
                  - name: local-persistent-storage
                    mountPath: /mnt
              volumes:
                - name: local-persistent-storage
                  persistentVolumeClaim:
                    claimName: local-storage
            ```
1. Trouble Shooting
    1. <https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#access-control>
1. Reference links:
    1. <https://lapee79.github.io/en/article/use-a-local-disk-by-local-volume-static-provisioner-in-kubernetes/>
    1. <https://medium0.com/alterway/kubernetes-local-static-provisioner-4c197e0f83ab>
    1. <https://blogs.oracle.com/cloud-infrastructure/post/using-local-persistent-volumes-with-container-engine-for-kubernetes>
    1. ***Dynamic provisioning of Kubernetes Local PVs*** <https://github.com/openebs/dynamic-localpv-provisioner>
    1. Using nfs <https://github.com/openebs/dynamic-nfs-provisioner>
    1. <https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner>
