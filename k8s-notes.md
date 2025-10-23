要查看 Kubernetes 集群中所有 Pod 的 IP 地址，可以使用 `kubectl` 命令结合相关参数查询。以下是常用的方法：

1. **简洁查看所有 Pod 的 IP**  
   使用 `kubectl get pods` 并指定输出格式，只显示名称和 IP：  
   ```bash
   kubectl get pods -o wide | awk '{print $1, $6}'
   ```  
   （`-o wide` 会显示额外信息，包括 IP；`awk` 用于提取名称和 IP 列）

2. **查看所有命名空间的 Pod IP**  
   如果需要包含所有命名空间（默认只看 `default` 命名空间），加上 `-A` 参数：  
   ```bash
   kubectl get pods -A -o wide | awk '{print $1, $2, $7}'
   ```  
   （此时列顺序变化，$1 是命名空间，$2 是 Pod 名称，$7 是 IP）

3. **更灵活的 JSON 输出过滤**  
   用 `jsonpath` 精确提取信息（适合脚本或复杂过滤）：  
   ```bash
   kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}{" "}{.metadata.name}{" "}{.status.podIP}{"\n"}{end}'
   ```  
   会输出 `命名空间 Pod名称 IP` 的格式，清晰区分不同命名空间的 Pod。

4. **查看单个 Pod 的详细 IP 信息**  
   如果需要某个 Pod 的详细网络信息（如容器 IP、主机 IP 等），可以描述该 Pod：  
   ```bash
   kubectl describe pod <pod名称> -n <命名空间> | grep IP
   ```

这些命令可以帮助你快速获取集群中所有 Pod 的 IP 地址，根据是否需要区分命名空间选择对应的方法即可。
