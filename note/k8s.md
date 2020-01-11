# pod

## nginx.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    resources:
        limits:
          cpu: "500m"
          memory: "128Mi"
```

## Pod 生命周期

- Pending: Pod 已经在 apiserver 中创建，但还没有调度到 Node 上面
- Running: Pod 已经调度到 Node 上面，所有容器都已经创建，并且至少有一个容器还在运行或者正在启动
- Succeeded: Pod 调度到 Node 上面后成功运行结束，并且不会重启
- Failed: Pod 调度到 Node 上面后至少有一个容器运行失败（即退出码不为 0 或者被系统终止）
- Unknonwn: 状态未知，通常是由于 apiserver 无法与 kubelet 通信导致

## RestartPolicy

- Always：只要退出就重启
- OnFailure：失败退出（exit code 不等于 0）时重启
- Never：只要退出就不再重启

## 镜像拉取策略 ImagePullPolicy

- Always：不管镜像是否存在都会进行一次拉取
- Never：不管镜像是否存在都不会进行拉取
- IfNotPresent：只有镜像不存在时，才会进行镜像拉取

### 注意：

- 默认为 IfNotPresent，但 :latest 标签的镜像默认为 Always。
- 拉取镜像时 docker 会进行校验，如果镜像中的 MD5 码没有变，则不会拉取镜像数据。
- 生产环境中应该尽量避免使用 :latest 标签，而开发环境中可以借助 :latest 标签自动拉取最新的镜像。

## 资源限制

- CPU 的单位是 CPU 个数，可以用 millicpu (m) 表示少于 1 个 CPU 的情况，如 500m = 500millicpu = 0.5cpu，而一个 CPU 相当于
  - AWS 上的一个 vCPU
  - GCP 上的一个 Core
  - Azure 上的一个 vCore
  - 物理机上开启超线程时的一个超线程
- 内存的单位则包括 E, P, T, G, M, K, Ei, Pi, Ti, Gi, Mi, Ki 等。

# Node

Node 是 Pod 真正运行的主机，可以是物理机，也可以是虚拟机。为了管理 Pod，每个 Node 节点上至少要运行 container runtime（比如 `docker` 或者 `rkt`）、`kubelet` 和 `kube-proxy` 服务。

## 每个 Node 都包括以下状态信息：

- 地址：包括 hostname、外网 IP 和内网 IP
- 条件（Condition）：包括 OutOfDisk、Ready、MemoryPressure 和 DiskPressure
- 容量（Capacity）：Node 上的可用资源，包括 CPU、内存和 Pod 总数
- 基本信息（Info）：包括内核版本、容器引擎版本、OS 类型等

## 维护模式

`kubectl cordon $NODENAME`

# Namespace

Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的 pods, services, replication controllers 和 deployments 等都是属于某一个 namespace 的（默认是 default），而 node, persistentVolumes 等则不属于任何 namespace。

# Service

Service 是应用服务的抽象，通过 labels 为应用提供负载均衡和服务发现。匹配 labels 的 Pod IP 和端口列表组成 endpoints，由 kube-proxy 负责将服务 IP 负载均衡到这些 endpoints 上。

每个 Service 都会自动分配一个 cluster IP（仅在集群内部可访问的虚拟地址）和 DNS 名，其他容器可以通过该地址或 DNS 来访问服务，而不需要了解后端容器的运行。

## nginx-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - port: 8078 # the port that this service should serve on
    name: http
    # the container on each pod to connect to, can be a name
    # (e.g. 'www') or a number (e.g. 80)
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
```

# Label

Label 是识别 Kubernetes 对象的标签，以 key/value 的方式附加到对象上（key 最长不能超过 63 字节，value 可以为空，也可以是不超过 253 字节的字符串）。

Label 不提供唯一性，并且实际上经常是很多对象（如 Pods）都使用相同的 label 来标志具体的应用。

Label 定义好后其他对象可以使用 Label Selector 来选择一组相同 label 的对象（比如 ReplicaSet 和 Service 用 label 来选择一组 Pod）。Label Selector 支持以下几种方式：

- 等式，如 app=nginx 和 env!=production
- 集合，如 env in (production, qa)
- 多个 label（它们之间是 AND 关系），如 app=nginx,env=test

# Annotations

Annotations 是 key/value 形式附加于对象的注解。不同于 Labels 用于标志和选择对象，Annotations 则是用来记录一些附加信息，用来辅助应用部署、安全策略以及调度策略等。比如 deployment 使用 annotations 来记录 rolling update 的状态。

# 使用 Volume

Pod 的生命周期通常比较短，只要出现了异常，就会创建一个新的 Pod 来代替它。那容器产生的数据呢？容器内的数据会随着 Pod 消亡而自动消失。Volume 就是为了持久化容器数据而生

## Persistent Volume

PersistentVolume (PV) 和 PersistentVolumeClaim (PVC) 提供了方便的持久化卷：PV 提供网络存储资源，而 PVC 请求存储资源。

## Volume 生命周期

1. Provisioning，即 PV 的创建，可以直接创建 PV（静态方式），也可以使用 StorageClass 动态创建
1. Binding，将 PV 分配给 PVC
1. Using，Pod 通过 PVC 使用该 Volume，并可以通过准入控制 StorageObjectInUseProtection 阻止删除正在使用的 PVC
1. Releasing，Pod 释放 Volume 并删除 PVC
1. Reclaiming，回收 PV，可以保留 PV 以便下次使用，也可以直接从云存储中删除
1. Deleting，删除 PV 并从云存储中删除后段存储

## Volume 的状态

- Available：可用
- Bound：已经分配给 PVC
- Released：PVC 解绑但还未执行回收策略
- Failed：发生错误

## PV 的访问模式（accessModes）

- ReadWriteOnce（RWO）：是最基本的方式，可读可写，但只支持被单个节点挂载。
- ReadOnlyMany（ROX）：可以以只读的方式被多个节点挂载。
- ReadWriteMany（RWX）：这种存储可以以读写的方式被多个节点共享。不是每一种存储都支持这三种方式，像共享方式，目前支持的还比较少，比较常用的是 NFS。在 PVC 绑定 PV 时通常根据两个条件来绑定，一个是存储的大小，另一个就是访问模式。

## PV 的回收策略（persistentVolumeReclaimPolicy，即 PVC 释放卷的时候 PV 该如何操作）

- Retain，不清理, 保留 Volume（需要手动清理）
- Recycle，删除数据，即 `rm -rf /thevolume/*`（只有 NFS 和 HostPath 支持）
- Delete，删除存储资源，比如删除 AWS EBS 卷（只有 AWS EBS, GCE PD, Azure Disk 和 Cinder 支持）

## 推荐配置

- 对于需要强 IO 隔离的场景，推荐使用整块磁盘作为 Volume
- 对于需要容量隔离的场景，推荐使用分区作为 Volume
- 避免在集群中重新创建同名的 Node（无法避免时需要先删除通过 Affinity 引用该 Node 的 PV）
- 对于文件系统类型的本地存储，推荐使用 UUID （如 ls -l /dev/disk/by-uuid）作为系统挂载点
- 对于无文件系统的块存储，推荐生成一个唯一 ID 作软链接（如 /dev/dis/by-id）。这可以保证 Volume 名字唯一，并不会与其他 Node 上面的同名 Volume 混淆

## redis-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-persistent-storage
      mountPath: /data/redis
  volumes:
  - name: redis-persistent-storage
    hostPath:
      path: /data/
```

## Kubernetes volume 支持的插件

- emptyDir
    如果 Pod 设置了 emptyDir 类型 Volume， Pod 被分配到 Node 上时候，会创建 emptyDir，只要 Pod 运行在 Node 上，emptyDir 都会存在（容器挂掉不会导致 emptyDir 丢失数据），但是如果 Pod 从 Node 上被删除（Pod 被删除，或者 Pod 发生迁移），emptyDir 也会被删除，并且永久丢失。
- hostPath
    hostPath 允许挂载 Node 上的文件系统到 Pod 里面去。如果 Pod 需要使用 Node 上的文件，可以使用 hostPath。
- gcePersistentDisk
    gcePersistentDisk 可以挂载 GCE 上的永久磁盘到容器，需要 Kubernetes 运行在 GCE 的 VM 中。
- awsElasticBlockStore
    awsElasticBlockStore 可以挂载 AWS 上的 EBS 盘到容器，需要 Kubernetes 运行在 AWS 的 EC2 上。
- nfs
    NFS 是 Network File System 的缩写，即网络文件系统。Kubernetes 中通过简单地配置就可以挂载 NFS 到 Pod 中，而 NFS 中的数据是可以永久保存的，同时 NFS 支持同时写操作。
- iscsi
- flocker
- glusterfs
- rbd
- cephfs
- gitRepo
    gitRepo volume 将 git 代码下拉到指定的容器路径中
- secret
- persistentVolumeClaim
- downwardAPI
- azureFileVolume
- vsphereVolume

### 使用 subPath

Pod 的多个容器使用同一个 Volume 时，subPath 非常有用

### FlexVolume

如果内置的这些 Volume 不满足要求，则可以使用 FlexVolume 实现自己的 Volume 插件。注意要把 volume plugin 放到 `/usr/libexec/kubernetes/kubelet-plugins/volume/exec/<vendor~driver>/<driver>`，plugin 要实现 `init/attach/detach/mount/umount` 等命令

### Projected Volume

Projected volume 将多个 Volume 源映射到同一个目录中，支持 secret、downwardAPI 和 configMap。

# 资源限制

# 健康检查

Kubernetes 作为一个面向应用的集群管理工具，需要确保容器在部署后确实处在正常的运行状态。Kubernetes 提供了两种探针（Probe，支持 exec、tcpSocket 和 http 方式）来探测容器的状态：

- LivenessProbe：探测应用是否处于健康状态，如果不健康则删除并重新创建容器
- ReadinessProbe：探测应用是否启动完成并且处于正常服务状态，如果不正常则不会接收来自 Kubernetes Service 的流量

# Kubernetes 主要由以下几个核心组件组成：

- etcd 保存了整个集群的状态；
  - 基本的 key-value 存储
  - 监听机制
  - key 的过期及续约机制，用于监控和服务发现
  - 原子 CAS 和 CAD，用于分布式锁和 leader 选举
- kube-apiserver 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制；
- kube-controller-manager 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
  由一系列的控制器组成
  - Replication Controller
  - Node Controller
  - CronJob Controller
  - Daemon Controller
  - Deployment Controller
  - Endpoint Controller
  - Garbage Collector
  - Namespace Controller
  - Job Controller
  - Pod AutoScaler
  - RelicaSet
  - Service Controller
  - ServiceAccount Controller
  - StatefulSet Controller
  - Volume Controller
  - Resource quota Controller
- kube-scheduler 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上；
  - 调度器需要充分考虑的因素:
    - 公平调度
    - 资源高效利用
    - QoS
    - affinity 和 anti-affinity
    - 数据本地化（data locality）
    - 内部负载干扰（inter-workload interference）
    - deadlines
  - Taints 和 tolerations
  Taints 和 tolerations 用于保证 Pod 不被调度到不合适的 Node 上，其中 Taint 应用于 Node 上，而 toleration 则应用于 Pod 上。
    - NoSchedule：新的 Pod 不调度到该 Node 上，不影响正在运行的 Pod
    - PreferNoSchedule：soft 版的 NoSchedule，尽量不调度到该 Node 上
    - NoExecute：新的 Pod 不调度到该 Node 上，并且删除（evict）已在运行的 Pod。Pod 可以增加一个时间（tolerationSeconds）
  - 优先级调度
  - 多调度器
  - 调度器扩展
- kubelet 负责维持容器的生命周期，同时也负责 Volume（CVI）和网络（CNI）的管理；
- Container runtime 负责镜像管理以及 Pod 和容器的真正运行（CRI），默认的容器运行时为 Docker；
- kube-proxy 负责为 Service 提供 cluster 内部的服务发现和负载均衡；

## 推荐的 Add-ons

- kube-dns 负责为整个集群提供 DNS 服务
- Ingress Controller 为服务提供外网入口
- Heapster 提供资源监控
- Dashboard 提供 GUI
- Federation 提供跨可用区的集群
- Fluentd-elasticsearch 提供集群日志采集、存储与查询

# 分层架构

- 核心层：Kubernetes 最核心的功能，对外提供 API 构建高层的应用，对内提供插件式应用执行环境
- 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS 解析等）
- 管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态 Provision 等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy 等）
- 接口层：kubectl 命令行工具、客户端 SDK 以及集群联邦
- 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴
  - Kubernetes 外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS 应用、ChatOps 等
  - Kubernetes 内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等

# API设计原则

1. 所有API应该是声明式的。
1. API对象是彼此互补而且可组合的。
1. 高层API以操作意图为基础设计。
1. 低层API根据高层API的控制需要设计。
1. 尽量避免简单封装，不要有在外部API无法显式知道的内部隐藏的机制。
1. API操作复杂度与对象数量成正比。
1. API对象状态不能依赖于网络连接状态。
1. 尽量避免让操作机制依赖于全局状态，因为在分布式系统中要保证全局状态的同步是非常困难的。

# 控制机制设计原则

- 控制逻辑应该只依赖于当前状态。
- 假设任何错误的可能，并做容错处理。
- 尽量避免复杂状态机，控制逻辑不要依赖无法监控的内部状态。
- 假设任何操作都可能被任何操作对象拒绝，甚至被错误解析。
- 每个模块都可以在出错后自动恢复。
- 每个模块都可以在必要时优雅地降级服务。

# 核心概念

- 复制控制器（Replication Controller，RC）
- 副本集（Replica Set，RS）
- 部署(Deployment)
- 服务（Service）
- 任务（Job）
- 后台支撑服务集（DaemonSet）
- 有状态服务集（StatefulSet）
- 集群联邦（Federation）
- 存储卷（Volume）
- 持久存储卷（Persistent Volume，PV）和持久存储卷声明（Persistent Volume Claim，PVC）
- 节点（Node）
- 密钥对象（Secret）
- 用户帐户（User Account）和服务帐户（Service Account）
- 名字空间（Namespace）
  - K8s集群初始有两个名字空间，分别是默认名字空间default和系统名字空间kube-system
- RBAC访问授权

# kubectl

## 常用命令格式

- 创建：kubectl run <name> --image=<image> 或者 kubectl create -f manifest.yaml
- 查询：kubectl get <resource>
- 更新 kubectl set 或者 kubectl patch
- 删除：kubectl delete <resource> <name> 或者 kubectl delete -f manifest.yaml
- 查询 Pod IP：kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'
- 容器内执行命令：kubectl exec -ti <pod-name> sh
- 容器日志：kubectl logs [-f] <pod-name>
- 导出服务：kubectl expose deploy <name> --port=80
- Base64 解码：`kubectl get secret SECRET -o go-template='{{ .data.KEY | base64decode }}'`
- 连接到一个正在运行的容器: kubectl attach
- 在容器内部执行命令: kubectl exec
- 文件拷贝: kubectl cp 支持从容器中拷贝，或者拷贝文件到容器中

# 命令行自动补全

## Linux 系统

```
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
```

## MacOS zsh

```
source <(kubectl completion zsh)
```

# Horizontal Pod Autoscaling (HPA)

## HPA 最佳实践

- 为容器配置 CPU Requests
- HPA 目标设置恰当，如设置 70% 给容器和应用预留 30% 的余量
- 保持 Pods 和 Nodes 健康（避免 Pod 频繁重建）
- 保证用户请求的负载均衡
- 使用 kubectl top node 和 kubectl top pod 查看资源使用情况

# ConfigMap

- 用作环境变量
- 用作命令行参数
- ConfigMap 必须在 Pod 引用它之前创建
- 使用 envFrom 时，将会自动忽略无效的键
- Pod 只能使用同一个命名空间内的 ConfigMap

# Ingress

# Resource Quotas

资源配额（Resource Quotas）是用来限制用户资源用量的一种机制。

- 资源配额应用在 Namespace 上，并且每个 Namespace 最多只能有一个 ResourceQuota 对象
- 开启计算资源配额后，创建容器时必须配置计算资源请求或限制（也可以用 LimitRange 设置默认值）
- 用户超额后禁止创建新的资源

# LimitRange

默认情况下，Kubernetes 中所有容器都没有任何 CPU 和内存限制。LimitRange 用来给 Namespace 增加一个资源限制，包括最小、最大和默认资源。

# Secret

Secret 解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者 Pod Spec 中。Secret 可以以 Volume 或者环境变量的方式使用。

# kubectl 安装

## Mac OSX

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
```

## Linux

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```

# 单机部署 minikube

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl

# install minikube
$ brew cask install minikube
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-hyperkit
$ sudo install -o root -g wheel -m 4755 docker-machine-driver-hyperkit /usr/local/bin/

# start minikube.
# http proxy is required in China
$ minikube start --docker-env HTTP_PROXY=http://proxy-ip:port --docker-env HTTPS_PROXY=http://proxy-ip:port --vm-driver=hyperkit
```

# Kubernetes 配置最佳实践

- 定义配置文件的时候，指定最新的稳定 API 版本。
- 在部署配置文件到集群之前应该保存在版本控制系统中。这样当需要的时候能够快速回滚，必要的时候也可以快速的创建集群。
- 使用 YAML 格式而不是 JSON 格式的配置文件。在大多数场景下它们都可以互换，但是 YAML 格式比 JSON 更友好。
- 尽量将相关的对象放在同一个配置文件里，这样比分成多个文件更容易管理。参考 [guestbook-all-in-one.yaml](https://github.com/kubernetes/examples/blob/master/guestbook/all-in-one/guestbook-all-in-one.yaml) 文件中的配置。
- 使用 kubectl 命令时指定配置文件目录。
- 不要指定不必要的默认配置，这样更容易保持配置文件简单并减少配置错误。
- 将资源对象的描述放在一个 annotation 中可以更好的内省。

## Services

- 通常最好在创建相关的 replication controllers 之前先创建 service。这样可以保证容器在启动时就配置了该服务的环境变量。对于新的应用，推荐通过服务的 DNS 名字来访问（而不是通过环境变量）。
- 除非有必要（如运行一个 node daemon），不要使用配置 hostPort 的 Pod（用来指定暴露在主机上的端口号）。当你给 Pod 绑定了一个 hostPort，该 Pod 会因为端口冲突很难调度。如果是为了调试目的来通过端口访问的话，你可以使用 kubectl proxy and apiserver proxy 或者 kubectl port-forward。你可使用 Service 来对外暴露服务。如果你确实需要将 pod 的端口暴露到主机上，考虑使用 NodePort service。
- 跟 hostPort 一样的原因，避免使用 hostNetwork。
- 如果你不需要 kube-proxy 的负载均衡的话，可以考虑使用使用 headless services（ClusterIP 为 None）。

# Kubernetes 日志

- Logstash（或者 Fluentd）负责收集日志
    除了 Fluentd 和 Logstash，还可以使用 Filebeat 来收集日志
- Elasticsearch 存储日志并提供搜索
- Kibana 负责日志查询和展示

# GPU

# GCP

## 虚拟私有网络（VPC）

```
gcloud compute networks create kubernetes-the-hard-way --mode custom

gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24

# 防火墙规则: 创建一个防火墙规则允许内部网路通过所有协议进行通信
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16

# 创建一个防火墙规则允许外部 SSH、ICMP 以及 HTTPS 等通信
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0

# list ipf
gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"

# 分配固定的 IP 地址, 被用来连接外部的负载平衡器至 Kubernetes API Servers
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)

# 验证 kubernetes-the-hard-way 固定 IP 地址已经在默认的 Compute Region 中创建出来
gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
```

## Kubernetes 控制节点

```
for i in 0 1 2; do
  gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
done

```

## Kubernetes 工作节点

```
for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
done
```

```
# list
gcloud compute instances list
```

## 配置 SSH

```
gcloud compute ssh controller-0
```

# note

- 每个API对象都有3大类属性：元数据metadata、规范spec和状态status。
- 所有的操作都是声明式（Declarative）的而不是命令式（Imperative）的。
- 目前K8s中的业务主要可以分为长期伺服型（long-running）、批处理型（batch）、节点后台支撑型（node-daemon）和有状态应用型（stateful application）；分别对应的小机器人控制器为Deployment、Job、DaemonSet和StatefulSet
- 有下面两组近义词；第一组是无状态（stateless）、牲畜（cattle）、无名（nameless）、可丢弃（disposable）；第二组是有状态（stateful）、宠物（pet）、有名（having name）、不可丢弃（non-disposable）。
- Etcd 源码下有个 tools/etcd-dump-logs，可以将 wal 日志 dump 成文本查看，可以协助分析 raft 协议。
