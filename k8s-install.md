> #### 作者：pengyl
>
> 官方网站：https://github.com/pengyongliang

kubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署，本文记录了本地通过VirtaulBox+centos7+kubeadm 部署集群的过程

```
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口 >
```

## 1. 安装要求

在开始之前，部署Kubernetes集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap分区

## 2. 基础环境准备
  - env config
     vm/centos7.x image
  - VirtaulBox
      install doc
      https://embeddedinventor.com/cant-ping-virtualbox-troubleshooting-guide-and-solution/
   - Understanding Common Networking Configurations
      https://www.virtualbox.org/manual/ch06.html#networkingmodes
  - vmware workstation（可选）
     https://www.centennialsoftwaresolutions.com/post/bridged-nat-host-only-or-custom-vmware

 ## 3. 准备环境
   
  ![kubernetesæ¶æå¾](https://blog-1252881505.cos.ap-beijing.myqcloud.com/k8s/single-master.jpg) 

| 角色       | IP            |
| ---------- | ------------- |
| k8s-master | 192.168.31.61 |
| k8s-node1  | 192.168.31.62 |
| k8s-node2  | 192.168.31.63 |


```
   set-hostname(every node)
      hostnamectl set-hostname xxxx
    stop firewalld(every node)
      systemctl stop firewalld
      systemctl disable firewalld
    swapoff
      swapoff -a
  config fstab(every node)
      1） vi /etc/fstab
      2） comment last line，eg
      #/dev/mapper/centos-swap swap                    swap    defaults        0 0
      
      3）reboot now
  ip_forward (every node)
       echo 1 > /proc/sys/net/ipv4/ip_forward
  disabled selinux(every node)
     sed -i 's/enforcing/disabled/' /etc/selinux/config
     swapoff -a  # 临时
  config hosts(every node)
       vi /etc/hosts 
       ip1 k8s-master
       ip2 k8s-node1
       ip3 k8s-node2
   config k8s.conf(every node)
       cat > /etc/sysctl.d/k8s.conf << EOF
       net.bridge.bridge-nf-call-ip6tables = 1
       net.bridge.bridge-nf-call-iptables = 1
       EOF
   install docker(every node)
       wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
       yum -y install docker-ce-18.06.1.ce-3.el7
       systemctl enable docker && systemctl start docker
        cat > /etc/docker/daemon.json << EOF
        {
          "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
        }
        EOF
   config kubernates.repo(every node)
       cat > /etc/yum.repos.d/kubernetes.repo << EOF
       [kubernetes]
       name=Kubernetes
       baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
       enabled=1
       gpgcheck=0
       repo_gpgcheck=0
       gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
       EOF
  install kubelet & kubeadm & kubectl tools(every node)
       yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
       systemctl enable kubelet
   kubeadm init( master node )
       kubeadm init   --apiserver-advertise-address=ip   --image-repository registry.aliyuncs.com/google_containers   --kubernetes-version v1.18.0   --service-cidr=10.96.0.0/12   --pod-network-cidr=10.244.0.0/16
       
       mkdir -p $HOME/.kube
       sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
       sudo chown $(id -u):$(id -g) $HOME/.kube/config
       
       kubectl get nodes
       
  install network plugin( master node )
       kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
       
  install kubernate dashboard( master node )
       
       kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
       kubectl get services
       kubectl get pods --all-namespaces -o wide
       kubectl create serviceaccount dashboard-admin -n kube-system
       kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
       --print admin token
       kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
       
  delete kubernate dashboard ( master node )
       kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

   # reset k8s 集群
   
     1. 主节点执行
     
      sudo kubeadm reset -f
      sudo rm -rf /etc/kubernetes/pki /var/lib/etcd /etc/cni/net.d ~/.kube/config
      sudo systemctl restart docker kubelet
  

    kubeadm init   --image-repository registry.aliyuncs.com/google_containers   --kubernetes-version v1.18.0   --service-cidr=10.96.0.0/12   -pod-network-cidr=10.244.0.0/16

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

    

    2. 从节点执行
       sudo kubeadm reset
       sudo rm -rf /etc/kubernetes/kubelet.conf
       sudo rm -rf /etc/kubernetes/pki/ca.crt
       sudo rm -rf /var/lib/kubelet/*
       sudo rm -rf /etc/cni/net.d
       sudo systemctl restart kubelet
       sudo kubeadm join 192.168.56.5:6443 --token yj57tm.92w6ket8nd1m57r8   --discovery-token-ca-cert-hash sha256:0932ab0bf2830ad285937268d313e3bc87c2b2061fe270b0af2e0d260ad12ca7
       
       
     
