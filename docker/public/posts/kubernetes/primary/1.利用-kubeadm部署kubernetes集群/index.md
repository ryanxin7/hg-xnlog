# 利用kubeadm部署kubernetes集群





## 1.kubeadm 部署工具



kubeadm 是kubernetes 集群全生命周期管理工具，可用于实现集群的部署、升级/降级及卸载等。

kubeadm 的核心工具是 **kubeadm init** 和 **kubeadm join** ，前者用于创建新的控制平面节点，后者则用于将节点快速连接到指定的控制平面。



{{< admonition tip "kubeadm init" true>}}



`kubeadm init`: 这个命令用于初始化一个新的 Kubernetes  集群的控制平面节点。

在运行这个命令后，它将自动生成一个初始的控制平面配置，创建必要的证书和密钥，以及启动必要的核心组件，如 etcd、API  Server、Controller Manager 和 Scheduler。



`kubeadm join`: 这个命令用于将工作节点快速连接到已经初始化的 Kubernetes 控制平面。

当你运行 `kubeadm init` 后，它会生成一个令牌 (token) 和一个哈希 (hash)，你可以将它们用于通过 `kubeadm join` 命令将其他节点加入到集群中。

这些节点将会成为集群的工作节点，与控制平面节点协同工作。



{{< /admonition >}}



Kubernetes 中的令牌 (**token**) 在集群中用于节点加入过程的身份认证和安全通信。这些令牌由 `kubeadm` 或管理员生成，并用于在新节点首次联系 Kubernetes API Server 时进行身份验证。



{{< admonition success "Kubernetes Token" true>}}



Kubernetes 令牌是集群加入过程中的关键组成部分，它们帮助确保新节点的身份得到验证，并确保集群的安全性。

1. **加入令牌 (Join Token)**: 在使用 `kubeadm init` 初始化控制平面节点后，它会生成一个加入令牌 (Join Token)。这个令牌包括一个令牌字符串和一个哈希值。新节点需要提供这个令牌和哈希值，以便加入到集群中。通过运行 `kubeadm join` 命令并提供正确的令牌，新节点可以成功加入集群。

   

2. **预共享密钥 (Pre-shared Key)**: 加入令牌的哈希值是通过使用预共享密钥计算生成的。这个密钥存储在控制平面节点上，新节点通过令牌中的哈希值和控制平面节点上的密钥进行匹配，从而实现身份验证。这确保了只有具有有效令牌和正确密钥的节点才能加入集群。

   

3. **有效期**: 令牌通常具有一定的有效期，过期后将无法使用。这是为了安全考虑，以确保令牌不会永久存在，从而降低潜在的风险。

   

4. **令牌轮换**: 在一些情况下，管理员可能需要轮换令牌，以提高集群的安全性。`kubeadm` 提供了命令来生成新的令牌并轮换现有令牌。

{{< /admonition >}}



其他几个可用的工具包括：

**kubeadm config**: 这个命令用于生成和修改 `kubeadm` 的配置文件，这些配置文件包括初始化集群和加入节点时使用的配置。通过 `kubeadm config` 命令，你可以生成初始化或加入节点所需的配置文件，然后对其进行自定义修改。这使得在不同的部署环境中更容易地配置 Kubernetes 集群。

例如，你可以使用 `kubeadm config print init-defaults` 命令来生成初始化集群所需的默认配置文件，并在需要时进行修改。

**kubeadm upgrade**: 这个命令用于执行 Kubernetes 集群的升级操作。Kubernetes 的升级通常涉及到控制平面组件 (Control Plane) 和工作节点 (Worker Node) 的升级。`kubeadm upgrade` 命令提供了一种升级路径，可用于在不中断生产工作负载的情况下将集群升级到新版本。

升级过程通常包括以下步骤：

- 升级控制平面组件。
- 升级 `kubeadm` 工具本身。
- 升级工作节点。

`kubeadm upgrade` 命令可用于管理这些升级步骤，使升级过程更加平滑。

**kubeadm reset**: 这个命令用于将节点恢复到初始状态，即将节点上的 Kubernetes 组件和配置删除，从而卸载 Kubernetes。这在需要重新部署或卸载节点时非常有用。运行 `kubeadm reset` 将清除节点上的所有 Kubernetes 配置和数据。

请注意，`kubeadm reset` 只用于卸载 Kubernetes，并不会卸载容器运行时、操作系统组件或其他可能在节点上存在的软件。







2.集群组件运行模式



3.kubeadm inti 工作流程



4.kubeadm join 工作流程

---

> 作者: [Ryan](https://github.com/ryanxin7)  
> URL: https://hg-xnlog.github.io/posts/kubernetes/primary/1.%E5%88%A9%E7%94%A8-kubeadm%E9%83%A8%E7%BD%B2kubernetes%E9%9B%86%E7%BE%A4/  

