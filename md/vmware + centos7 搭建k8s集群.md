vmware + centos7 搭建k8s集群

- 安装centos，克隆两台（一台用作master节点， 一台用作node节点， ip地址设为固定ip，用hostnamectl命令设置hostname）

  ![VMware-centos固定ip](vmware%20+%20centos7%20%E6%90%AD%E5%BB%BAk8s%E9%9B%86%E7%BE%A4.assets/VMware-centos%E5%9B%BA%E5%AE%9Aip.jpg)

  ```
  // 修改主机名
  hostnamectl set-hostname 
  
  // 配置固定ip
  vi /etc/sysconfig/network-scripts/ifcfg-ens33
  BOOTPROTO="static"
  IPADDR="192.168.153.128"
  NETMAST="255.255.255.0"
  GATEWAY="192.168.153.2"
  DNS1="192.168.153.2"
  ONBOOT="yes"
  
  ```

  

- 安装docker

  1. 安装Device Mapper所需的软件包（Device Mapper是 Linux2.6 内核中支持逻辑卷管理的通用设备映射机制，它为实现用于存储资源管理的块设备驱动提供了一个高度模块化的内核架构。）

     sudo yum install -y yum-utils device-mapper-persistent-data lvm2

  2. 设置docker源

     sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 

  3. 安装docker ce

     ```
     sudo yum update -y && sudo yum install -y \
       containerd.io-1.2.13 \
       docker-ce-19.03.11 \
       docker-ce-cli-19.03.11
     ```

  4. 设置docker daemon

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
  
  5. 重启docker 设置开机启动
     ```
     systemctl daemon-reload
     systemctl restart docker
     systemctl enable docker
     
     ```
  
- 安装k8s

  1. 安装前系统配置

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

  2. 安装kubeadm

     ```
     # 配置源
     cat <<EOF > /etc/yum.repos.d/kubernetes.repo
     [kubernetes]
     name=Kubernetes
     baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
     enabled=1
     gpgcheck=0
     repo_gpgcheck=0
     gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
            http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
     EOF
     # 安装
     yum install -y kubelet-1.19.13-0 kubeadm-1.19.13-0 kubectl-1.19.13-0 --disableexcludes=kubernetes
     systemctl enable --now kubelet
     
     ```
  
     
  
  3. 拉取docker镜像(只在master节点执行，但是其他节点也需要kube-proxy镜像)
  
     ```
     # 查看镜像列表
     [root@k8s-master k8s]# kubeadm config images list
     
     I0727 15:41:17.797252   55674 version.go:255] remote version is much newer: v1.21.3; falling back to: stable-1.19
     W0727 15:41:19.767306   55674 configset.go:348] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
     k8s.gcr.io/kube-apiserver:v1.19.13
     k8s.gcr.io/kube-controller-manager:v1.19.13
     k8s.gcr.io/kube-scheduler:v1.19.13
     k8s.gcr.io/kube-proxy:v1.19.13
     k8s.gcr.io/pause:3.2
     k8s.gcr.io/etcd:3.4.13-0
     k8s.gcr.io/coredns:1.7.0
     
     # 编写拉取脚本
     vi /tmp/kubeadm.sh
     #!/bin/bash
     set -e
     KUBE_VERSION=v1.19.13
     KUBE_PAUSE_VERSION=3.2
     ETCD_VERSION=3.4.13-0
     CORE_DNS_VERSION=1.7.0
     GCR_URL=k8s.gcr.io
     ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/google_containers
     images=(kube-proxy:${KUBE_VERSION}
     kube-scheduler:${KUBE_VERSION}
     kube-controller-manager:${KUBE_VERSION}
     kube-apiserver:${KUBE_VERSION}
     pause:${KUBE_PAUSE_VERSION}
     etcd:${ETCD_VERSION}
     coredns:${CORE_DNS_VERSION})
     for imageName in ${images[@]} ; do
             docker pull $ALIYUN_URL/$imageName
             docker tag  $ALIYUN_URL/$imageName $GCR_URL/$imageName
             docker rmi $ALIYUN_URL/$imageName
     done
     
     # 执行脚本
     sh /tmp/kubeadm.sh
     ```
  
  4.  初始化集群
  
     ```
     # 官网用法: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
     # apiserver-advertise-address: master节点ip
     # service-cidr: service 虚拟ip范围
     # pod-network-cidr: pod 虚拟ip范围
     kubeadm init --kubernetes-version=1.19.13 \
     --apiserver-advertise-address=192.168.240.129 \
     --service-cidr=10.96.0.0/12 \
     --pod-network-cidr=10.244.0.0/16
     
     看到下面的信息就初始化成功了,在master上执行下面的命令拷贝admin.conf到用户目录下
     ############
     Your Kubernetes control-plane has initialized successfully!
     
     To start using your cluster, you need to run the following as a regular user:
     
       mkdir -p $HOME/.kube
       sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
       sudo chown $(id -u):$(id -g) $HOME/.kube/config
     
     You should now deploy a pod network to the cluster.
     Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
       https://kubernetes.io/docs/concepts/cluster-administration/addons/
     
     Then you can join any number of worker nodes by running the following on each as root:
     
     kubeadm join 192.168.240.138:6443 --token 2mmo8t.oi5451u2826rrmg3 \
         --discovery-token-ca-cert-hash sha256:25cd0agaeagaga84f27yqiwytyt897aga01937915c5fd5e2c4da795b998facac
     ```
  
  5.  添加其他node节点到集群
  
     ```
     # 在其他节点执行下面的命令
     kubeadm join 192.168.240.138:6443 --token 2mmo8t.oi5451u2826rrmg3 \
         --discovery-token-ca-cert-hash sha256:25cd0agaeagaga84f27yqiwytyt897aga01937915c5fd5e2c4da795b998facac
     ```
  
     将其他节点加到集群后，在master节点就可以查看到其他节点了但是状态是NotReady。这是由于 kubeadm 默认不提供网络插件，还要安装网络插件
  
     ![image-20210728164002330](vmware%20+%20centos7%20%E6%90%AD%E5%BB%BAk8s%E9%9B%86%E7%BE%A4.assets/image-20210728164002330.png)
  
  6. 安装网络插件
  
     - 插件简介 
  
       > - ACI 通过 Cisco ACI 提供集成的容器网络和安全网络。
       > - Calico 是一个安全的 L3 网络和网络策略驱动。
       > - Canal 结合 Flannel 和 Calico，提供网络和网络策略。
       > - Cilium 是一个 L3 网络和网络策略插件，能够透明的实施 HTTP/API/L7 策略。 同时支持路由（routing）和覆盖/封装（overlay/encapsulation）模式。
       > - CNI-Genie 使 Kubernetes 无缝连接到一种 CNI 插件， 例如：Flannel、Calico、Canal、Romana 或者 Weave。
       > - Contiv 为多种用例提供可配置网络（使用 BGP 的原生 L3，使用 vxlan 的覆盖网络， 经典 L2 和 Cisco-SDN/ACI）和丰富的策略框架。Contiv 项目完全开源。 安装工具同时提供基于和不基于 kubeadm 的安装选项。
       > - 基于 Tungsten Fabric 的 Contrail 是一个开源的多云网络虚拟化和策略管理平台，Contrail 和 Tungsten Fabric 与业务流程系统 （例如 Kubernetes、OpenShift、OpenStack和Mesos）集成在一起， 为虚拟机、容器或 Pod 以及裸机工作负载提供了隔离模式。
       > - Flannel 是一个可以用于 Kubernetes 的 overlay 网络提供者。
       > - Knitter 是为 kubernetes 提供复合网络解决方案的网络组件。
       > - Multus 是一个多插件，可在 Kubernetes 中提供多种网络支持， 以支持所有 CNI 插件（例如 Calico，Cilium，Contiv，Flannel）， 而且包含了在 Kubernetes 中基于 SRIOV、DPDK、OVS-DPDK 和 - VPP 的工作负载。
       > - OVN-Kubernetes 是一个 Kubernetes 网络驱动， 基于 OVN（Open Virtual Network）实现，是从 Open vSwitch (OVS) 项目衍生出来的虚拟网络实现。 OVN-Kubernetes 为 Kubernetes 提供基于覆盖网络的网络实现，包括一个基于 OVS 实现的负载均衡器 和网络策略。
       >   OVN4NFV-K8S-Plugin 是一个基于 OVN 的 CNI 控制器插件，提供基于云原生的服务功能链条（Service Function Chaining，SFC）、多种 OVN 覆盖 网络、动态子网创建、动态虚拟网络创建、VLAN 驱动网络、直接驱动网络，并且可以 驳接其他的多网络插件，适用于基于边缘的、多集群联网的云原生工作负载。
       > - NSX-T 容器插件（NCP） 提供了 VMware NSX-T 与容器协调器（例如 Kubernetes）之间的集成，以及 NSX-T 与基于容器的 CaaS / PaaS 平台（例如关键容器服务（PKS）和 OpenShift）之间的集成。
       > - Nuage 是一个 SDN 平台，可在 Kubernetes Pods 和非 Kubernetes 环境之间提供基于策略的联网，并具有可视化和安全监控。
       > - Romana 是一个 pod 网络的第三层解决方案，并支持 NetworkPolicy API。 Kubeadm add-on 安装细节可以在这里找到。
       > - Weave Net 提供在网络分组两端参与工作的网络和网络策略，并且不需要额外的数据库。
  
     - 这里选择calico插件安装([官网](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises))
  
       小于50个node，使用curl https://docs.projectcalico.org/manifests/calico.yaml -O。
       如果是超过50个node使用curl https://docs.projectcalico.org/manifests/calico-typha.yaml -o calico.yaml
  
       ```
       #下载yaml文件
       curl https://docs.projectcalico.org/manifests/calico.yaml -O
       # 安装
       kubectl apply -f calico.yaml
       #查看pod的情况
       kubectl get po -A --watch
       ```
  
       查看部署状态
  
       ![image-20210728165516266](vmware%20+%20centos7%20%E6%90%AD%E5%BB%BAk8s%E9%9B%86%E7%BE%A4.assets/image-20210728165516266.png)





​     