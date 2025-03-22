#kubectl get pods bookstore-microservices-domain-account-859d5f5d74-hflnh -n bookstore-microservices  -o yaml |grep containerID

#kubectl get pods -o wide -A

# [root@k8s-node1 ~]# docker inspect -f {{.State.Pid}} 71a3011d631f9b09ca3111b8e763d58d08bcd1a3d14abee1db7afaf9ab1a6bbc
11812

# nsenter -n -t 11812

# tcpdump -n -w account.pod.cap

#download dumpfile to local
scp root@192.168.56.10:/root/account.pod.cap ~/Downloads/
