
## 使用到的一些资源

### `NetworkPolicy`


![](attachments/Pasted%20image%2020250225202348.png)

- 通过 `kubectl client` 创建 network policy 资源
- calico 的 policy -controller 监听 network policy 资源，获取后写入calico的etcd数据库
- node 上的calico-felix 从etcd数据库获取policy资源，调用iptables做相应配置

### `Role`(`ClusterRole`) && `RoleBinding`(`ClusterRoleBinding`)

该Role含有的权限，通过`RoleBinding`之后就会赋予对应的User, Group, Service Account相应的权限

![](attachments/Pasted%20image%2020250225210010.png)





## ctr

kube-prometheus镜像加载使用命令 `ctr -n k8s.io image import sipaas_V5.0.02.030.36.tar` 需要指定命名空间为k8s.io