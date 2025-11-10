
使用 `ip addr` 能查看各个网卡的命名空间


```bash
140: cali61c9dff40df@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-65aa19f2-48e0-06fc-c6a0-5b8944caef98
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link 
       valid_lft forever preferred_lft forever
141: calib27783a9fdc@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-c8d72641-64c2-aa2b-dd67-a67dbdb705e3
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link 
       valid_lft forever preferred_lft forever
142: calie985949cd78@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-b9167225-5a29-83f1-6a7d-193810291312
    inet6 fe80::ecee:eeff:feee:eeee/64 scope link 
       valid_lft forever preferred_lft forever
```


## 查看对应Pod属于哪个网卡

### 网卡对查看方法

```bash
kubectl exec -ti alertmanager-main-0 -n base-services -- sh -c 'cat /sys/class/net/eth0/iflink'
43
```
如果对应的pod里面不能执行命令，可以使用如下方式查找
```bash
# 查看pod的容器
crictl ps | grep ysp-ui-lzwd2
e81fa0489d6ed       71d9be26110de       9 minutes ago       Running             my-container                0                   3eb95c3598168       ysp-ui-lzwd2
# 根据容器ID查看进程PID信息
crictl inspect -o yaml e81fa0489d6ed | grep pid
          pid: 1
  pid: 2764398
      - type: pid
# ls -l /proc/{pid}/ns/net  根据PID查看网卡信息
ls -l /proc/2764398/ns/net
# 通过/proc/<pid>/ns/net可以访问容器的网络命名空间。为了方便操作，可以创建符号链接到/var/run/netns/ 目录
ln -s /proc/2764398/ns/net /var/run/netns/pod_netns
# 查看网络信息，可以看到这里使用的是和宿主机的的64号网卡对应
ip netns exec pod_netns ip addr
...
4: eth0@if64: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 3a:e1:83:31:3d:57 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 100.66.218.186/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::38e1:83ff:fe31:3d57/64 scope link 
       valid_lft forever preferred_lft forever
# 使用 ip link show | grep "^64:" 可以查看宿主机上对应的网卡， 如果容器没有创建网卡，那么默认就是都是4编号
ip link show | grep "^64:"
64: cali70d3ca51123@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default 
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns pod_netns
```

### 路由表查看方
1. 进入到Pod内部查看ip地址
```bash
/prometheus $ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
4: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 66:76:a5:8c:fb:5f brd ff:ff:ff:ff:ff:ff
    inet 100.70.244.246/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6476:a5ff:fe8c:fb5f/64 scope link 
       valid_lft forever preferred_lft forever
```
2. 进入到对应主机查看路由
```bash
[root@k8smaster-1 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.0.1     0.0.0.0         UG    100    0        0 enp0s3
100.70.244.192  0.0.0.0         255.255.255.192 U     0      0        0 *
100.70.244.208  0.0.0.0         255.255.255.255 UH    0      0        0 cali57c605ce2cc
100.70.244.215  0.0.0.0         255.255.255.255 UH    0      0        0 cali20668973fa0
100.70.244.219  0.0.0.0         255.255.255.255 UH    0      0        0 cali09fe7018ef7
100.70.244.220  0.0.0.0         255.255.255.255 UH    0      0        0 cali1f45ed39910
100.70.244.221  0.0.0.0         255.255.255.255 UH    0      0        0 cali989de1080ad
100.70.244.222  0.0.0.0         255.255.255.255 UH    0      0        0 cali5459fa44f4f
100.70.244.224  0.0.0.0         255.255.255.255 UH    0      0        0 cali1176d9871d1
100.70.244.227  0.0.0.0         255.255.255.255 UH    0      0        0 cali1e6c2b85682
100.70.244.236  0.0.0.0         255.255.255.255 UH    0      0        0 cali3dbf08c6d60
100.70.244.237  0.0.0.0         255.255.255.255 UH    0      0        0 cali0e4f05f74fc
100.70.244.238  0.0.0.0         255.255.255.255 UH    0      0        0 calic2eeda9bb38
100.70.244.246  0.0.0.0         255.255.255.255 UH    0      0        0 caliee699845661
192.168.0.0     0.0.0.0         255.255.255.0   U     100    0        0 enp0s3

# 或者
[root@k8smaster-1 ~]# ip r s 
default via 192.168.0.1 dev enp0s3 proto static metric 100 
blackhole 100.70.244.192/26 proto bird 
100.70.244.208 dev cali57c605ce2cc scope link 
100.70.244.215 dev cali20668973fa0 scope link 
100.70.244.219 dev cali09fe7018ef7 scope link 
100.70.244.220 dev cali1f45ed39910 scope link 
100.70.244.221 dev cali989de1080ad scope link 
100.70.244.222 dev cali5459fa44f4f scope link 
100.70.244.224 dev cali1176d9871d1 scope link 
100.70.244.227 dev cali1e6c2b85682 scope link 
100.70.244.236 dev cali3dbf08c6d60 scope link 
100.70.244.237 dev cali0e4f05f74fc scope link 
100.70.244.238 dev calic2eeda9bb38 scope link 
100.70.244.246 dev caliee699845661 scope link 
192.168.0.0/24 dev enp0s3 proto kernel scope link src 192.168.0.122 metric 100 
```

经过另个对比可以看到网络是经过 `100.70.244.246` 发送出去的





## 常用命令

```bash
$ kubectl get ippool -o yaml
apiVersion: v1
items:
  ...
  kind: IPPool
  spec:
    allowedUses:
    - Workload
    - Tunnel
    cidr: 100.64.0.0/10
    ipipMode: Always
    natOutgoing: true
    nodeSelector: all()
    vxlanMode: Never
  ...
```





