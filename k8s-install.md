 # install kubernates
   ## 1. env config
     vm/centos7.x image
   ## 2. VirtaulBox
      install doc
      https://embeddedinventor.com/cant-ping-virtualbox-troubleshooting-guide-and-solution/
      Understanding Common Networking Configurations
      https://www.virtualbox.org/manual/ch06.html#networkingmodes
   ## 3. vmware workstation
     https://www.centennialsoftwaresolutions.com/post/bridged-nat-host-only-or-custom-vmware

   ## 4. 准备环境
   

    | 角色        | IP            |
    | ---------- | ------------- |
    | k8s-master | 192.168.56.5  |
    | k8s-node1  | 192.168.56.10 |
    | k8s-node2  | 192.168.56.11 |


   ## 5. set-hostname(every node)
      hostnamectl set-hostname xxxx
   ## 6. stop firewalld(every node)
      systemctl stop firewalld
      systemctl disable firewalld
   ## 7. swapoff
      swapoff -a
   ## 8. config fstab(every node)
      1） vi /etc/fstab
      2） comment last line，eg
      #/dev/mapper/centos-swap swap                    swap    defaults        0 0
      
      3）reboot now
   ## 9. ip_forward (every node)
       echo 1 > /proc/sys/net/ipv4/ip_forward
   ## 10. disabled selinux(every node)
     sed -i 's/enforcing/disabled/' /etc/selinux/config
     swapoff -a  # 临时
   ## 11. config hosts(every node)
       vi /etc/hosts 
       ip1 k8s-master
       ip2 k8s-node1
       ip3 k8s-node2
   ## 12. config k8s.conf(every node)
       cat > /etc/sysctl.d/k8s.conf << EOF
       net.bridge.bridge-nf-call-ip6tables = 1
       net.bridge.bridge-nf-call-iptables = 1
       EOF
   ## 13. install docker(every node)
       wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
       yum -y install docker-ce-18.06.1.ce-3.el7
       systemctl enable docker && systemctl start docker
        cat > /etc/docker/daemon.json << EOF
        {
          "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
        }
        EOF
   ## 14. config kubernates.repo(every node)
       cat > /etc/yum.repos.d/kubernetes.repo << EOF
       [kubernetes]
       name=Kubernetes
       baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
       enabled=1
       gpgcheck=0
       repo_gpgcheck=0
       gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
       EOF
   ###15. install kubelet & kubeadm & kubectl tools(every node)
       yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
       systemctl enable kubelet
   ## 16. kubeadm init( master node )
       kubeadm init   --apiserver-advertise-address=ip   --image-repository registry.aliyuncs.com/google_containers   --kubernetes-version v1.18.0   --service-cidr=10.96.0.0/12   --pod-network-cidr=10.244.0.0/16
       
       mkdir -p $HOME/.kube
       sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
       sudo chown $(id -u):$(id -g) $HOME/.kube/config
       
       kubectl get nodes
       
   ## 17. install network plugin( master node )
       kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
       
   ## 18. if  install kubernate dashboard( master node )
       kubectl apply -f https://github.com/pengyongliang/k8s/blob/b9472652d5b84647b2957087d8c88d8b33b06797/recommended.yaml
       kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
       kubectl get services
       kubectl get pods --all-namespaces -o wide
       kubectl create serviceaccount dashboard-admin -n kube-system
       kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
       --print admin token
       kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
       
   ## 19. delete kubernate dashboard ( master node )
       sudo kubectl delete deployment kubernetes-dashboard --namespace=kubernetes-dashboard 
       sudo kubectl delete service kubernetes-dashboard  --namespace=kubernetes-dashboard 
       sudo kubectl delete service dashboard-metrics-scraper  --namespace=kubernetes-dashboard 
       sudo kubectl delete role.rbac.authorization.k8s.io kubernetes-dashboard --namespace=kubernetes-dashboard 
       sudo kubectl delete clusterrole.rbac.authorization.k8s.io kubernetes-dashboard --namespace=kubernetes-dashboard
       sudo kubectl delete rolebinding.rbac.authorization.k8s.io kubernetes-dashboard --namespace=kubernetes-dashboard
       sudo kubectl delete clusterrolebinding.rbac.authorization.k8s.io kubernetes-dashboard --namespace=kubernetes-dashboard
       sudo kubectl delete deployment.apps kubernetes-dashboard --namespace=kubernetes-dashboard
       sudo kubectl delete deployment.apps dashboard-metrics-scraper --namespace=kubernetes-dashboard
       sudo kubectl delete sa kubernetes-dashboard --namespace=kubernetes-dashboard 
       sudo kubectl delete secret kubernetes-dashboard-certs --namespace=kubernetes-dashboard
       sudo kubectl delete secret kubernetes-dashboard-csrf --namespace=kubernetes-dashboard
       sudo kubectl delete secret kubernetes-dashboard-key-holder --namespace=kubernetes-dashboard
       sudo kubectl delete namespace kubernetes-dashboard 
       sudo kubectl delete configmap kubernetes-dashboard-settings

   ## 20. kubectl delete -f /home/recommended-custom.yml (master node)


   # reset k8s 集群
     ## 1. 主节点执行
       
      sudo kubeadm reset -f
      sudo rm -rf /etc/kubernetes/pki /var/lib/etcd /etc/cni/net.d ~/.kube/config
      sudo systemctl restart docker kubelet
  
       kubeadm init   --apiserver-advertise-address=ip   --image-repository registry.aliyuncs.com/google_containers   --kubernetes-version v1.18.0
  --service-cidr=10.96.0.0/12   --pod-network-cidr=10.244.0.0/16
    kubeadm init   --apiserver-advertise-address=ipv4   --image-repository registry.aliyuncs.com/google_containers   --kubernetes-version v1.18.0   --service-cidr=10.96.0.0/12   --pod-network-cidr=10.244.0.0/16
    kubeadm init   --image-repository registry.aliyuncs.com/google_containers   --kubernetes-version v1.18.0   --service-cidr=10.96.0.0/12   -p
od-network-cidr=10.244.0.0/16
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

    ## 2. 从节点执行
       sudo kubeadm reset
       sudo rm -rf /etc/kubernetes/kubelet.conf
       sudo rm -rf /etc/kubernetes/pki/ca.crt
       sudo rm -rf /var/lib/kubelet/*
       sudo rm -rf /etc/cni/net.d
       sudo systemctl restart kubelet
       sudo kubeadm join 192.168.56.5:6443 --token yj57tm.92w6ket8nd1m57r8   --discovery-token-ca-cert-hash sha256:0932ab0bf2830ad285937268d313e3bc87c2b2061fe270b0af2e0d260ad12ca7
     
