# 核心概念
## Pod
k8s最小的调度单位。一个Pod可以包含一个或多个紧密协作的容器（共享网络和存储）。
Pod 通常不会被直接创建，而是通过 Deployment 等控制器来管理。当节点发生故障时，控制器会在其他可用节点上重新创建 Pod。

## Node
运行 Pod 的物理机或虚拟机。
必要组件包括 kubelet、容器运行时（如 containerd 或 CRI-O）和 kube-proxy；
### 容器状态
容器状态用来描述节点的当前状态。现在，其中包含三个信息：
#### 主机IP
#### 节点状态
节点的状态通过一组条件（Conditions）来描述。
主要条件包括 `Ready`（kubelet 健康且可以接收 Pod）
`MemoryPressure`（内存不足）
`DiskPressure`（磁盘不足）和 `PIDPressure`（进程数过多）等。其中 `Ready` 条件最为关键：值为 `True` 表示节点健康可调度，`False` 表示节点异常，`Unknown` 表示节点控制器超过一定时间未收到心跳。被标记为 `Unknown` 较长时间的节点，其上的 Pod 会被驱逐重新调度到其他节点。
### 节点管理
```bash
$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
control-plane  Ready    control-plane   10d   v1.36.0
worker-1       Ready    <none>          10d   v1.36.0
worker-2       Ready    <none>          10d   v1.36.0
```
每个节点的详细信息以如下结构保存：
```yaml
apiVersion: v1
kind: Node
metadata:
  name: worker-1
  labels:
    kubernetes.io/os: linux
status:
  capacity:
    cpu: "4"
    memory: 8Gi
  conditions:
    - type: Ready
      status: "True"
```
### 节点控制器
在 Kubernetes 控制平面中，节点控制器 (Node Controller) 负责管理节点的生命周期，主要包含：
- 集群范围内节点状态同步
- 单节点生命周期管理
节点控制器会持续监控节点的健康状态。当节点变为不可达时，控制器会等待一个超时期限，然后将该节点上的 Pod 标记为失败，并触发重新调度。可以使用 `kubectl` 来管理节点，例如标记节点为不可调度或排空节点上的工作负载：
```bash
## 标记节点为不可调度
$ kubectl cordon worker-1

## 排空节点上的 Pod
$ kubectl drain worker-1 --ignore-daemonsets
```
## Development
定义应用的期望状态 (如：需要 3 个副本，镜像版本为 v1)。K8s 会持续确保当前状态符合期望状态。
## Service
定义一组 Pod 的访问策略。提供稳定的 Cluster IP 和 DNS 名称，负责负载均衡。
## Namespace
用于多租户资源隔离。
## 和Docker的概念对比
![](assets/K8s/file-20260623231311300.png)
# 架构
Kubernetes 也是 C/S 架构，由 **Control Plane (控制平面)** 和 **Worker Node (工作节点)** 组成：

- **Control Plane**：负责决策 (API Server，Scheduler，Controller Manager，etcd)
    
- **Worker Node**：负责干活 (Kubelet，Kube-proxy，Container Runtime)