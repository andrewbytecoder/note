




## netns
和nsenter类似，使用ip的子命令 `netns` 也能实现进入到具体网卡的网络空间进行抓包，而且可以直接使用宿主机上的工具执行网络相关的命令

### 使用宿主机上的ip addr命令，查看容器内部的网络地址
```bash
# 查看网络地址，特别是 link-netns
[root@worker2 core]# ip addr 
90: 83ef0e73e1fd745@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue master ovs-system state UP group default 
    link/ether 9a:8b:65:16:aa:ac brd ff:ff:ff:ff:ff:ff link-netns ac05efd5-e66d-480b-a459-280b6cd29d5a
    inet6 fe80::988b:65ff:fe16:aaac/64 scope link 
       valid_lft forever preferred_lft forever
91: 469a6b286dd1b48@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue master ovs-system state UP group default 
    link/ether 3e:76:ac:99:1c:64 brd ff:ff:ff:ff:ff:ff link-netns e74465a5-5bbb-4fd5-be81-5a032092be8a
    inet6 fe80::3c76:acff:fe99:1c64/64 scope link 
       valid_lft forever preferred_lft forever

[root@worker2 core]# ip netns exec  e74465a5-5bbb-4fd5-be81-5a032092be8a ip addr 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if91: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP group default qlen 1000
    link/ether 0a:58:0a:e1:28:0f brd ff:ff:ff:ff:ff:ff link-netns 3f3d1c2e-222a-482c-a9f5-4cb3f280c012
    inet 10.225.40.15/21 brd 10.225.47.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::858:aff:fee1:280f/64 scope link 
       valid_lft forever preferred_lft forever
```

### 使用本地的tcpdump可执行文件抓取容器内的网络流量
```bash
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




