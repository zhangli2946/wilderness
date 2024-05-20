### 🙂 cka notes
- 排序 pod 输出名字
  - 操作
    - `kubectl -A top pod -l name=cpu-loader --sort-by=cpu`
      1. -A 指定所有命名空间
      2. -l 指定 label, 格式 key=value
      3. --sort-by 指定排序参数 cpu/memory
- 创建 RBAC
  > 仔细分辨要求创建的是 clusterrole 还是 role
  > rolebinding 还是 clusterrolebinding 
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
- 集群升级
  - 操作
    - `kubectl get nodes && kubectl cordon master01 && kubectl drain master01 --ignore-daemonsets=false --delete-emptydir-data`
      1. 确认要升级的节点 
      2. 标记其进入维护状态
      3. 清理在其节点上运行的 pod
    - 执行操作
      ```bash 
      ssh master01                         # 远程到要升级的节点上
      kubeadm version                      # 确认现用 kubeadm 版本
      sudo -i                              # 获取管理员权限
      apt-get update                       # 更新 apt 索引
      apt-cache show kubeadm | grep 1.28.1 # 检查 kubeadm 版本
      apt-get install kubeadm=1.28.1-00    # 安装心版本
      kubeadm version                      # 确认升级 kubeadm 版本
      kubeadm upgrade plan                 # 验证升级计划，会显示很多可升级的版本，我们关注题目要求升级到的那个版本。 
      kubeadm upgrade apply v1.28.1        \
        --etcd-upgrade=false               # 排除 etcd，升级其他的，提示时，输入 y。大约要等 5 分钟。
      apt-get install kubelet=1.28.1-00    # 升级 kubelet
      systemctl restart kubelet            # 重启 kubelet
      kubelet --version                    # 通过 cli 版本 确认升级
      kubectl uncordon master01            # 恢复 master01 调度
      kubectl get nodes                    # 检查 node 恢复情况
      ```
  - etcd 升级
    - 操作
      - 执行操作
        ```bash
        export ETCDCTL_API=3
        etcdctl --endpoints=https://127.0.0.1:2379              \
                --cacert="/opt/KUIN00601/ca.crt"                \
                --cert="/opt/KUIN00601/etcd-client.crt"         \
                --key="/opt/KUIN00601/etcd-client.key"          \
                snapshot save /var/lib/backup/etcd-snapshot.db
        etcdctl snapshot status /var/lib/backup/etcd-snapshot.db -wtable
        ```
      - 执行操作
        ```bash
        export ETCDCTL_API=3
        sudo mv /var/lib/etcd /var/lib/etcd.bak                       # 备份原始数据
        etcdctl --data-dir=/var/lib/etcd                        \
                snapshot restore /data/backup/etcd-snapshot-previous.db
        sudo chown -R etcd:etcd /var/lib/etcd
        sudo systemctl start etcd                                     # 重启
        ```
- 配置网络策略
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-port-from-namespace
    namespace: my-app
  spec:
    podSelector: {}
    policyTypes:
      - Ingress
    ingress:
      - from:
        - namespaceSelector:
            matchLabels:
              name: echo
        ports:
         - protocol: TCP
           port: 9000
  ```
  - 解释
    - metadata.name 约定 网络策略的名称
    - metadata.namespace 约定 网络策略所属的命名空间
    - spec.podSelector 约定 网络策略针对命名空间中的哪些 pod 生效 "{}" 表示所有
    - spec.policyTypes 只允许 "Ingress" "Egress" 枚举值
- 为 pod 创建一个svc
  - 操作
    - 增补 pod 配置
      ```yaml
      spec:
        containers:
        - ports:
          - name: http
            containerPort: 80
            protocol: TCP
      ```
    - 创建 service 发布服务
      ```bash
      kubectl expose TYPE NAME         
            [--name=name]                             # Service 对象的名字    
            [--port=port]                             # Service 对外暴露的端口
            [--protocol=TCP|UDP|SCTP]                 # Service 对外提供的协议
            [--target-port=number-or-name]            # TYPE/NAME 的 containerPort 名字或者端口号
            [--type=type]                             # ClusterIP, NodePort, LoadBalancer, or ExternalName
      ```
- 多 container pod
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: kucc8
  spec:
    containers:
    - name: nginx
      image: nginx
    - name: consul
      image: consul
  ```
- 创建 pv
  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: task-pv-volume
    labels:
      type: local
  spec:
    storageClassName: manual
    capacity:
      storage: 1Gi
    accessModes:
      - ReadWriteMany
    hostPath:
      path: "/srv/app-config" 
  ```
- 创建 pvc 挂在到pod 并修改 要求的容量
  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  volumes:
  - name: use-as
    persistentVolumeClaim:
     claimName: pv-volume
  containers:
  - name: web-server
    image: "nginx:1.16"
    volumeMounts:
    - name: use-as
      mountPath: /usr/share/nginx/html
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 10Mi
```