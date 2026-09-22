
## 查看pod网卡

### 通过ovn网络管理node查看网卡信息

ovn网络在ocp中是openshift-ovn-kubernetes中的ovnkube-node-xxxxx中进行管理，可以进入到Pod内部查看集群网络的路由表
```bash
[ysp-dc3@localhost ~]$ kubectl exec -it -n openshift-ovn-kubernetes ovnkube-node-7bcqr -- sh
sh-5.1# 
sh-5.1# ovn-nbctl lr-route-list ovn_cluster_router
IPv4 Routes
Route Table <main>:
               100.64.0.2                100.88.0.2 dst-ip
               100.64.0.3                100.88.0.3 dst-ip
               100.64.0.4                100.88.0.4 dst-ip
               100.64.0.5                100.88.0.5 dst-ip
               100.64.0.6                100.64.0.6 dst-ip
               100.64.0.7                100.88.0.7 dst-ip
            10.226.0.0/21                100.88.0.3 dst-ip
            10.226.8.0/21                100.88.0.2 dst-ip
           10.226.16.0/21                100.88.0.4 dst-ip
           10.226.24.0/21                100.88.0.5 dst-ip
           10.226.40.0/21                100.88.0.7 dst-ip
           10.226.32.0/21               10.226.32.2 src-ip
            10.226.0.0/16                100.64.0.6 src-ip
```

iface-id 的组合形式为 `<namespace>_<pod name>`，当然如果命令 `ovn-nbctl list Logical_Switch_Port` 也能列出 iface-id的详细内容

```bash
sh-5.1# ovs-vsctl --data=bare --no-heading   --columns=name,external_ids   find Interface external_ids:iface-id="openshift-monitoring_prometheus-k8s-1"
15ccc127be7ca3e
attached_mac=0a:58:0a:e2:20:08 iface-id=openshift-monitoring_prometheus-k8s-1 iface-id-ver=d01c9434-b02a-45e9-bcd5-bb8de48fe085 ip_addresses=10.226.32.8/21 ovn-installed=true ovn-installed-ts=1788485939765 pod-if-name=eth0 sandbox=15ccc127be7ca3e4d4772230db52cf8e4597058c55cc922989fb2fc269e0839b
```

这里的 15ccc127be7ca3e 就是物理机上的网卡名，该网卡和pod内部的eth0组成一个网卡对
```bash
[core@worker2 ~]$ ifconfig|grep 15ccc127be7ca3e
15ccc127be7ca3e: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1400
```
### 当然可以输出网卡更加详细的信息
```bash
sh-5.1# ovs-vsctl find Interface external_ids:iface-id=openshift-monitoring_prometheus-k8s-1
_uuid               : ab942def-268b-43c1-bab4-ebbd29cd1fab
admin_state         : up
bfd                 : {}
bfd_status          : {}
cfm_fault           : []
cfm_fault_status    : []
cfm_flap_count      : []
cfm_health          : []
cfm_mpid            : []
cfm_remote_mpids    : []
cfm_remote_opstate  : []
duplex              : full
error               : []
external_ids        : {attached_mac="0a:58:0a:e2:20:08", iface-id=openshift-monitoring_prometheus-k8s-1, iface-id-ver="d01c9434-b02a-45e9-bcd5-bb8de48fe085", ip_addresses="10.226.32.8/21", ovn-installed="true", ovn-installed-ts="1788485939765", pod-if-name=eth0, sandbox="15ccc127be7ca3e4d4772230db52cf8e4597058c55cc922989fb2fc269e0839b"}
ifindex             : 25
ingress_policing_burst: 0
ingress_policing_kpkts_burst: 0
ingress_policing_kpkts_rate: 0
ingress_policing_rate: 0
lacp_current        : []
link_resets         : 0
link_speed          : 10000000000
link_state          : up
lldp                : {}
mac                 : []
mac_in_use          : "02:16:51:31:d3:5a"
mtu                 : 1400
mtu_request         : []
name                : "15ccc127be7ca3e"   # 这里的name就是物理主机上的网卡
ofport              : 15
ofport_request      : []
options             : {}
other_config        : {}
statistics          : {collisions=0, rx_bytes=38456318318, rx_crc_err=0, rx_dropped=0, rx_errors=0, rx_frame_err=0, rx_missed_errors=0, rx_multicast_packets=0, rx_over_err=0, rx_packets=220130091, tx_bytes=535911453817, tx_dropped=0, tx_errors=0, tx_packets=204619013, upcall_errors=0, upcall_packets=2526715}
status              : {driver_name=veth, driver_version="1.0", firmware_version=""}
type                : ""
```


如果只想要网卡名字可以过滤信息进行输出
```bash
sh-5.1# ovs-vsctl --data=bare --no-heading --columns=name   find Interface external_ids:iface-id=openshift-monitoring_prometheus-k8s-1
15ccc127be7ca3e
```

### 使用本地的tcpdump可执行文件抓取容器内的网络流量

```bash
# 查看网络地址，特别是 link-netns
[root@worker2 core]# ip addr | grep 469a6b286dd1b48 -A 10
91: 469a6b286dd1b48@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue master ovs-system state UP group default 
    link/ether 3e:76:ac:99:1c:64 brd ff:ff:ff:ff:ff:ff link-netns e74465a5-5bbb-4fd5-be81-5a032092be8a
    inet6 fe80::3c76:acff:fe99:1c64/64 scope link 
       valid_lft forever preferred_lft forever
#  link-netns e74465a5-5bbb-4fd5-be81-5a032092be8a   网卡UUID
[root@worker2 core]# ip netns exec  e74465a5-5bbb-4fd5-be81-5a032092be8a ./tcpdump
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
12:28:21.759565 IP worker2.z2.ameidc2.com.36118 > 10.225.32.73.etcd-client: Flags [P.], seq 766323013:766323037, ack 1279897928, win 510, options [nop,nop,TS val 3845428254 ecr 99831668], length 24
12:28:21.760123 IP 10.225.32.73.etcd-client > worker2.z2.ameidc2.com.36118: Flags [P.], seq 1:31, ack 24, win 505, options [nop,nop,TS val 99835173 ecr 3845428254], length 30
12:28:21.760132 IP worker2.z2.ameidc2.com.36118 > 10.225.32.73.etcd-client: Flags [.], ack 31, win 510, options [nop,nop,TS val 3845428254 ecr 99835173], length 0
12:28:21.760237 IP worker2.z2.ameidc2.com.36118 > 10.225.32.73.etcd-client: Flags [P.], seq 24:41, ack 31, win 510, options [nop,nop,TS val 3845428254 ecr 99835173], length 17
12:28:21.760769 IP 10.225.32.73.etcd-client > worker2.z2.ameidc2.com.36118: Flags [P.], seq 31:86, ack 41, win 505, options [nop,nop,TS val 99835174 ecr 3845428254], length 55
12:28:21.760881 IP worker2.z2.ameidc2.com.36118 > 10.225.32.73.etcd-client: Flags [P.], seq 41:71, ack 86, win 510, options [nop,nop,TS val 3845428255 ecr 99835174], length 30
12:28:21.761247 IP 10.225.32.73.etcd-client > worker2.z2.ameidc2.com.36118: Flags [P.], seq 86:103, ack 71, win 505, options [nop,nop,TS val 99835174 ecr 3845428255], length 17
12:28:21.801294 IP worker2.z2.ameidc2.com.36118 > 10.225.32.73.etcd-client: Flags [.], ack 103, win 510, options [nop,nop,TS val 3845428296 ecr 99835174], length 0
12:28:21.827792 IP worker2.z2.ameidc2.com.57640 > 10.161.46.81.domain: 58855+ PTR? 73.32.225.10.in-addr.arpa. (43)
```



### 查看pod网卡的详细信息

`ovn-nbctl list Logical_Switch_Port`

```bash
sh-5.1# ovn-nbctl list Logical_Switch_Port
_uuid               : 838db740-f68b-4124-a29a-3d5d4224b054
addresses           : ["0a:58:0a:e2:20:06 10.226.32.6"]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : []
external_ids        : {namespace=openshift-monitoring, pod="true"}
ha_chassis_group    : []
mirror_rules        : []
name                : openshift-monitoring_thanos-querier-874cc4b5d-bg2c8
options             : {iface-id-ver="13846e43-7583-484b-8f01-5459d9ce461a", requested-chassis="9fc7171e-471e-4b8e-a539-761a6736ef9e"}
parent_name         : []
peer                : []
port_security       : ["0a:58:0a:e2:20:06 10.226.32.6"]
tag                 : []
tag_request         : []
type                : ""
up                  : true

_uuid               : 5d190540-046d-4e1f-a6d9-cb2f6155c5f9
addresses           : ["0a:58:64:58:00:02 100.88.0.2/16"]
dhcpv4_options      : []
dhcpv6_options      : []
dynamic_addresses   : []
enabled             : []
external_ids        : {node=master1.z3.ameidc3.com}
ha_chassis_group    : []
mirror_rules        : []
name                : tstor-master1.z3.ameidc3.com
options             : {requested-chassis="93cf7a52-078a-4a10-92b3-53fde6cd2bb1", requested-tnl-key="2"}
parent_name         : []
peer                : []
port_security       : []
tag                 : []
tag_request         : []
type                : remote
up                  : true

...
```

### 查看端口从哪个桥出去的
```bash
ovs-vsctl iface-to-br <iface>
ovs-vsctl port-to-br <port>
```

```bash
sh-5.1# ovs-vsctl iface-to-br 15ccc127be7ca3e
br-int
```