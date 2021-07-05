# Join Worker Node

## Install K8s on Worker Node

1.  Using Config_system
1.  Update apt cache
    ```bash
    sudo apt-get update -y
    ```
1.  Install Docker
    ```bash
    sudo apt install -y docker.io=19.03.8-0ubuntu1 containerd=1.3.3-0ubuntu2

    sudo systemctl unmask docker.service
    sudo systemctl unmask docker.socket
    sudo systemctl start docker.service
    sudo systemctl status docker
    ```
1.  Config Docker driver
    ```bash
    sudo vi /etc/docker/daemon.json

    {
      "exec-opts": ["native.cgroupdriver=systemd"],
      "log-driver": "json-file",
      "log-opts": {
        "max-size": "100m"
      },
      "storage-driver": "overlay2"
    }
    ```
1.  Add a new repo for kubernetes
    ```bash
    sudo vi /etc/apt/sources.list.d/kubernetes.list

    deb http://apt.kubernetes.io/ kubernetes-xenial main
    ```
1.  Add a GPG key for the packages
    ```bash
    sudo -i

    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    ```
1.  Update with the new repo declared
    ```bash
    sudo apt-get update
    ```
1.  Install the software
    ```bash
    apt list -a |grep kuber
    apt-cache madison kubelet
    sudo apt-get install -y kubeadm=1.19.0-00 kubelet=1.19.0-00 kubectl=1.19.0-00
    sudo apt-mark hold kubelet kubeadm kubectl
    ```
1. Open firewall on k8s-master to accept connection from Worker node
   ```bash
   sudo ufw allow from 10.0.0.50 comment "Worker-10.0.0.50"
   ```
1. Open firewall on Worker Node to accept connection from k8s-master
   ```bash
   sudo ufw allow from 10.0.0.10 comment "k8s-master"
   ```
1. Add hosts k8s-master
   ```bash
   sudo vi /etc/hosts
   127.0.0.1 localhost
   10.0.0.10 k8s-master
   10.0.0.50 worker-10-0-0-50
   ```
1. Add hosts worker 
   ```bash
   sudo vi /etc/hosts
   127.0.0.1 localhost
   10.0.0.10 k8s-master
   10.0.0.50 worker-10-0-0-50
   ```
1. Create a token for joining worker node
   ```bash
   kubeadm token create --print-join-command

   kubeadm token list # check token
   ```
1. On Worker Node, using kubeadm cli to join worker node to k8s cluster
   ```bash
   sudo -i

   kubeadm join k8s-master:6443 --token kmx10g.o7de6j6v78qqkyjc --discovery-token-ca-cert-hash sha256:454b1661e0ccfe88736686f0ea5d756738f22381ebddd3a53ebf213bcfbc1523
   ```
1. On k8s-master Node, using kubectl check clusters
   ```bash
   sudo vi /etc/hosts
   10.0.0.50 worker-10.0.0.50

   kubectl get nodes
   
   NAME               STATUS   ROLES    AGE   VERSION
   k8s-master         Ready    master   28h   v1.19.0
   worker-10-0-0-50   Ready    <none>   66s   v1.19.0
   ```
