### 🙂 cka notes
- 排序 pod 输出名字
  - 操作
    - `kubectl -A top pod -l name=cpu-loader --sort-by=cpu`
      1. -A 指定所有命名空间
      2. -l 指定 label, 格式 key=value
      3. --sort-by 指定排序参数 cpu/memory
- 创建 RBAC
  - 操作
    - `kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployments,daemonsets,statefulsets`
      1. 创建一个 clusterrole, 以 deployment-clusterrole 命名
      2. --verb create:  赋予角色 创建权限 
      3. --resource deployments,daemons,statefulsets: 准予角色 操作的资源 deployments, daemons, statefulsets
    - `kubectl -n app-team1 create serviceaccount cicd-token`
      1. 在 app-team1 中, 创建 service account, 以 cicd-token 命名
    - `kubectl -n app-team1 create rolebinding cicd-token --serviceaccount=app-team1:cicd-token --clusterrole=deployment-clusterrole`
      1. clusterrolebinding 是不需要指定命名空间的, rolebinding 属于确定的命名空间
      2. 在 app-team1 中, 创建一个 rolebinding, 以 cicd-token 命名
      3. 指定 servicecount 是 app-team1:cicd-token
      4. 指定 clusterrole 是 deployment-clusterrole
  - 检查
    - kubectl -n app-team1 describe rolebinding cicd-token
- 扩容 deployments 副本数
  - 操作
    - `kubectl scale deployment presentation --replicas=4`
      1. scale 名为 presentation 的 deployment 资源 
      2. --replicas=4 指定 副本数为 4
  - 检查
    - `kubectl get po -owide --no-headers | grep presentation | wc -l `
- 调度 pod 到 特定节点上
  - 操作
    - `kubectl edit po xxxx`
      ```diff
      + - nodeSelector:
      +     disk: ssd
      ```
- 检查可用节点数量
  - 操作
    - `kubectl describe nodes | grep -i Taints | grep -vc NoSchedule`
- 查看 pod 日志
  - 操作
    - `kubectl logs foo | grep "RLIMIT_NOFILE" > /opt/KUTR00101/foo`
- 维护节点
  - 操作
    - `kubectl crodon node02 && kubectl drain node02 --ignore-daemonsets --delete-emptydir-data`
- 集群 kubelet 排除故障 (重启)
  - 操作
    - `systemctl enable kubelet && systemctl start kubelet`
  - 检查
    - `kubectl get nodes`

- yaml 题 还没写的