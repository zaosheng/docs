## rke搭建k8s集群



- 安装前准备

  - 准备几台可以相互ping通的机器(如vmware centos虚拟机 可以准备好一台 然后克隆 配置静态IP)

    ```
    vi /etc/sysconfig/network-scripts/ifcfg-ens33
    BOOTPROTO="static"
    ONBOOT="yes"
    IPADDR="192.168.153.140"
    PREFIX="24"
    GATEWAY="192.168.153.2"
    DNS1="192.168.153.2"
    DNS2=8.8.8.8
    ```

  - 系统配置

    ```
    # 关闭防火墙
    systemctl stop firewalld && systemctl disable firewalld
    
    
    # 关闭selinux
    setenforce 0
    sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config\
    
    # 关闭swap
    swapoff -a
    sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
    
    # 配置iptables的ACCEPT规则
    iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
    
    # 设置系统参数
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    
    # 加载配置
    sysctl --system
    
    ```

  - 安装docker
    1. 安装Device Mapper所需的软件包（Device Mapper是 Linux2.6 内核中支持逻辑卷管理的通用设备映射机制，它为实现用于存储资源管理的块设备驱动提供了一个高度模块化的内核架构。）

       ```
       sudo yum install -y yum-utils device-mapper-persistent-data lvm2
       ```

       

    2. 配置kubernetes源

          ```
          cat <<EOF > /etc/yum.repos.d/kubernetes.repo
          [kubernetes]
          name=Kubernetes
          baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
          enabled=1
          gpgcheck=1
          repo_gpgcheck=1
          gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
          EOF
          ```
    
          
    
    3. 设置docker源
    
       ```
       sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
       ```
    
       
    
    4. 安装docker ce
    
       ```
       sudo yum update -y && sudo yum install -y \
         containerd.io-1.2.13 \
         docker-ce-19.03.11 \
         docker-ce-cli-19.03.11
       ```
    
       
    
    5. 设置docker daemon
    
         ```
         cat <<EOF | sudo tee /etc/docker/daemon.json
         {
           "exec-opts": ["native.cgroupdriver=systemd"],
           "log-driver": "json-file",
           "log-opts": {
             "max-size": "100m"
           },
           "storage-driver": "overlay2",
           "storage-opts": [
             "overlay2.override_kernel_check=true"
           ]
         }
         EOF
         
         ```
    
    6. 重启docker 设置开机启动
         ```
         systemctl daemon-reload
         systemctl restart docker
         systemctl enable docker
         
         ```
    
    7. 添加用户
    
          ```
          # 添加用户 xxx
          useradd xxx
          passwd xxx
          # 将用户加入docker组
          usermod -aG docker xxx
          # 给用户sudo权限
          vi /etc/sudoers
          找到这一行："root ALL=(ALL) ALL"，
          在下面添加"xxx ALL=(ALL) ALL"
          # centos使用sudo时免密码
          解除/etc/sudoers中下面的注释：
          #%wheel ALL=(ALL)NOPASSWD:ALL
          
          #ubuntu使用sudo时免密码
          vi /etc/sudoers
          %sudo   ALL=(ALL:ALL) NOPASSWD:ALL
          %admin ALL=(ALL) NOPASSWD:ALL
          ```

- 克隆虚拟机，配置其他几台虚拟机的ip

- 安装k8s

  - 配置免密登录，在需要装rke的机器上执行以下操作：

    1. 切换用户到创建的用户XXX

       ```
       su - xxx
       ```

    2. ssh-keygen \#三次回车

    3. ssh-copy-id 用户名@机器IP  #输入yes和密码 ，  所有节点都要拷贝

    4. ssh 用户名@机器IP  #测试登录

  - 下载rke

    ```
    # 地址 https://github.com/rancher/rke/releases
    mv rke_darwin-amd64 rke-v1.2.11
    chmod +x rke-v1.2.11
    ```

  - 编写 cluster.yaml 文件

    ```
    vi cluster.yaml
    
    nodes:
      - address: 192.168.153.140
        user: rke-user
        role: [controlplane,etcd]
      - address: 192.168.153.141
        user: rke-user
        role: [etcd,worker]
      - address: 192.168.153.142
        user: rke-user
        role: [etcd,worker]
    ```

  - 安装(幂等操作)

    ```
    ./rke-v1.2.11 up --config cluster.yaml
    ```

    完成后当前目录会生成`kube_config_cluster.yml`文件和`cluster.rkestate`文件。

  - 安装kubectl

    1. 下载kubectl

        ```
        # 下载最新版
        curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
        
        #下载指定版本：https://kubernetes.io/docs/tasks/tools/install-kubectl/
        ```
        
    2.  拷贝kubectl到用户目录
    
        ```
        chmod +x ./kubectl
        sudo mv ./kubectl /usr/local/bin/kubectl
        ```

    3.  拷贝kube_config_cluster.yml 到用户目录
    
        ```
        cp ./kube_config_cluster.yml  ~/.kube/config
        
        ```
    
  - 错误处理
  
  - etcd health check failed
  
    ```
      INFO[0044] [etcd] Successfully started etcd plane.. Checking etcd cluster health
      WARN[0140] [etcd] host [192.168.153.140] failed to check etcd health: failed to get /health for host [192.168.153.140]: Get "https://192.168.153.140:2379/health": remote error: tls: internal error
      WARN[0234] [etcd] host [192.168.153.141] failed to check etcd health: failed to get /health for host [192.168.153.141]: Get "https://192.168.153.141:2379/health": remote error: tls: internal error
      ```
  
    执行下面命令后重新执行rke up 或者执行rke remove命令：
  
    ```
      sudo rm -rf /etc/kubernetes/ /var/lib/kubelet/ /var/lib/etcd/
      docker rm -f $(docker ps -qa)
      ```
  
    
