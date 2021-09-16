# Installation K8S

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
        "max-size": "20m",
        "max-file": "5"
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
1.  We will use Calico as a network plugin
    ```bash
    wget https://docs.projectcalico.org/manifests/calico.yaml
    ```
1.  Update Network range ip address for pods
    ```bash
    vi calico.yaml
    
    # The default IPv4 pool to create on startup if none exists. Pod IPs will be
    # chosen from this range. Changing this value after installation will have
    # no effect. This should fall within `--cluster-cidr`.
    - name: CALICO_IPV4POOL_CIDR
      value: "172.16.0.0/16"    
    ```
1.  Create a configuration file for the cluster
    ```bash
    sudo -i
    vi kubeadm-config.yaml

    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterConfiguration
    kubernetesVersion: 1.19.0
    controlPlaneEndpoint: "k8s-master:6443"
    networking:
      podSubnet: 172.16.0.0/16
    ```
1.  Initialize the master
    ```bash
    sudo -i
    kubeadm init --config=kubeadm-config.yaml --upload-certs | tee kubeadm-init.out
    ```
1.  Copy kubectl config
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
1.  Apply the network plugin configuration
    ```bash
    kubectl apply -f calico.yaml
    ```
1.  We will enable bash auto-completion
    ```bash
    sudo apt-get install bash-completion -y
    echo "source <(kubectl completion bash)" >> ~/.bashrc
    source ~/.bashrc
    ```
1.  Test by describing the node again
    ```bash
    kubectl get nodes

    NAME         STATUS   ROLES    AGE    VERSION
    k8s-master   Ready    master   118m   v1.19.0
    ```
