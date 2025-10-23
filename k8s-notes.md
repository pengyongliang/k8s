## 查看所有命名空间下的pod ip
```
kubectl get pods -A -o wide | awk '{print $1, $2, $7}'
```
