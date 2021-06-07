 # install kubernates
   ### env config
     vm/centos7.x image
   ### set-hostname
      hostnamectl set-hostname xxxx
   ### stop firewalld
      systemctl stop firewalld
      systemctl disable firewalld
   ### swapoff
      swapoff -a
   ### config fstab
      vi /etc/fstab
      exclude last line
      reboot now
   ### disabled selinux
     sed -i 's/enforcing/disabled/' /etc/selinux/config
   ### config hosts
       vi /etc/hosts 
       ip1 k8s-master
       ip2 k8s-node1
       ip3 k8s-node2
   ### config k8s.conf
       cat > /etc/sysctl.d/k8s.conf << EOF
       net.bridge.bridge-nf-call-ip6tables = 1
       net.bridge.bridge-nf-call-iptables = 1
       EOF
   ### install docker
       wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
       yum -y install docker-ce-18.06.1.ce-3.el7
       systemctl enable docker && systemctl start docker
        cat > /etc/docker/daemon.json << EOF
        {
          "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
        }
        EOF
   ### config kubernates.repo
       cat > /etc/yum.repos.d/kubernetes.repo << EOF
       [kubernetes]
       name=Kubernetes
       baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
       enabled=1
       gpgcheck=0
       repo_gpgcheck=0
       gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
       EOF
   ### install kubelet & kubeadm & kubectl tools
       yum install -y kubelet-1.18.0 kubeadm-1.17.0 kubectl-1.18.0
       systemctl enable kubelet
   ### kubeadm init( run script on every node )
       kubeadm init   --apiserver-advertise-address=ip   --image-repository registry.aliyuncs.com/google_containers   --kubernetes-version v1.18.0   --service-cidr=10.96.0.0/12   --pod-network-cidr=10.244.0.0/16
       
       mkdir -p $HOME/.kube
       sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
       sudo chown $(id -u):$(id -g) $HOME/.kube/config
       
       kubectl get nodes
   ### install network plugin
       kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   ### install kubernate dashboard
       kubectl apply -f https://github.com/pengyongliang/k8s/blob/b9472652d5b84647b2957087d8c88d8b33b06797/kube-flannel.yml
       kubectl get services
       kubectl get pods --all-namespaces -o wide
       kubectl create serviceaccount dashboard-admin -n kube-system
       kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
       --print admin token
       kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
