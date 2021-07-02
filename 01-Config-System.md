# System Configuration

1.  OS ubuntu 
    ```bash
    hadn@k8s-master:~$ cat /etc/lsb-release 
    DISTRIB_ID=Ubuntu
    DISTRIB_RELEASE=20.04
    DISTRIB_CODENAME=focal
    DISTRIB_DESCRIPTION="Ubuntu 20.04.2 LTS"
    ```
1.  Create user with sudo permission
    -   sudo vi  /etc/sudoers.d/student
        ```bash
        student ALL=(ALL) NOPASSWD: ALL
        ```
    -   sudo chmod 440 /etc/sudoers.d/student
1.  Disable swap
    -   sudo swapoff -v /swap.img
    -   sudo rm /swap.img
    -   sudo vi /etc/fstab
        ```bash
        #/swapfile swap swap defaults 0 0   
        ```
1.  Enable firewall
    -   sudo ufw enable
1.  Config file descriptor limit
    -   sudo vi /etc/security/limits.conf
        ```bash
        * hard nofile 100000
        * soft nofile 100000
        ```
    -   sudo vi /etc/systemd/system.conf
        ```bash
        DefaultLimitNOFILE=100000
        ```
1.  Set up iptables see bridged traffic
    -   sudo vi /etc/modules-load.d/k8s-overlay.conf
        ```bash
        overlay

        sudo lsmod |grep overlay # check enable/disable
        ```
    -   sudo vi /etc/modules-load.d/k8s.conf
        ```bash
        br_netfilter

        sudo lsmod |grep br_netfilter # check enable/disable
        ```
    -   sudo vi /etc/sysctl.d/k8s.conf
        ```bash
        net.bridge.bridge-nf-call-iptables = 1
        net.ipv4.ip_forward = 1
        net.bridge.bridge-nf-call-ip6tables = 1


        sudo sysctl -a |grep net.bridge # check
        sudo sysctl -a |grep net.ipv4.ip_forward # check
        ```
1.  Setup /etc/hosts file
    ```bash
    10.0.0.10 k8s-master
    ```
1.  Restart server
    -   sudo reboot
