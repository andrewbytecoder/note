## 查看bird协议类型

```bash
> kubectl exec -n calico-system calico-node-hmv7z -c calico-node -- birdcl show protocols
BIRD v0.3.3+birdv1.6.8 ready.
name     proto    table    state  since       info
static1  Static   master   up     01:34:32    
kernel1  Kernel   master   up     01:34:32    
device1  Device   master   up     01:34:32    
direct1  Direct   master   up     01:34:32    
Node_10_10_10_1 BGP      master   up     01:36:29    Established   
```

## bird查问题的三个角度

```bash
birdcl show route                     # 我有哪些路由
birdcl show route protocol Node_xxx   # 从这个 peer 学来的
birdcl show route export Node_xxx all # 我准备发给这个 peer 的（最重要）
```


### 查看bird里面的路由表
bird里面全部的路由表
```bash
> kubectl exec -n calico-system calico-node-hmv7z -c calico-node -- birdcl show route
BIRD v0.3.3+birdv1.6.8 ready.
0.0.0.0/0          via 172.21.0.1 on eth0 [kernel1 01:34:33] * (10)
10.10.10.0/24      dev eth1 [direct1 01:34:32] * (240)
                   via 10.10.10.1 on eth1 [Node_10_10_10_1 01:36:29] (100/0) [AS65001i]
172.16.0.240/28    blackhole [static1 01:47:56] * (200)
192.168.81.64/26   via 10.10.10.11 on eth1 [kernel1 01:34:35] * (10)
192.168.81.64/32   via 10.10.10.11 on eth1 [kernel1 01:34:35] * (10)
192.168.23.65/32   via 10.10.10.10 on eth1 [kernel1 01:34:35] * (10)
192.168.23.64/26   via 10.10.10.10 on eth1 [kernel1 01:34:35] * (10)
192.168.164.129/32 dev cali09577ea09cf [kernel1 01:34:33] * (10)
192.168.164.128/26 blackhole [static1 01:34:32] * (200)
                   blackhole [kernel1 01:34:35] (10)
192.168.164.128/32 dev vxlan.calico [direct1 01:34:36] * (240)
192.168.164.131/32 dev cali18d0019ad26 [kernel1 01:34:40] * (10)
192.168.164.130/32 dev cali0e77f3392c7 [kernel1 01:34:38] * (10)
192.168.164.133/32 dev calia8997c65485 [kernel1 01:34:41] * (10)
192.168.164.132/32 dev cali749f367983a [kernel1 01:34:40] * (10)
192.168.164.135/32 dev cali372d418400e [kernel1 01:34:45] * (10)
192.168.164.134/32 dev cali27d4dc902cd [kernel1 01:34:44] * (10)
192.168.164.137/32 dev caliac97b059081 [kernel1 01:34:47] * (10)
192.168.164.136/32 dev cali78fb2e18cf8 [kernel1 01:34:46] * (10)
192.168.164.139/32 dev cali2e1fce4af24 [kernel1 02:01:59] * (10)
192.168.164.138/32 dev cali24521941f5a [kernel1 01:36:29] * (10)
172.21.0.0/16      dev eth0 [direct1 01:34:32] * (240)
```

### bird从指定peer学到了哪些路由信息
```bash
> kubectl exec -n calico-system calico-node-hmv7z -c calico-node -- birdcl show route protocol Node_10_10_10_1
BIRD v0.3.3+birdv1.6.8 ready.
10.10.10.0/24      via 10.10.10.1 on eth1 [Node_10_10_10_1 01:36:29] (100/0) [AS65001i]
```

### bird向指定peer发送了哪些路由信息

```bash
# 查看 BIRD 在 **导出（export）给 BGP peer `Node_10_10_10_1`**​ 时，这条路由的完整属性
# calico 里面 命名规则  Node_<对端节点IP，点号换成下>
# Node_10_10_10_1 = 和 IP 为 10.10.10.1 的节点 / BGP peer 的会话
> kubectl exec -n calico-system calico-node-hmv7z -c calico-node -- birdcl show route export Node_10_10_10_1 all
BIRD v0.3.3+birdv1.6.8 ready.
172.16.0.240/28    blackhole [static1 01:47:56] * (200)
        Type: static unicast univ
        BGP.origin: IGP
        BGP.as_path: 65010
        BGP.next_hop: 10.10.10.12
        BGP.local_pref: 100
```






















