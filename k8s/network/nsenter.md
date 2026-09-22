核心作用，在宿主机上进入到，某个进程的namespace里使用宿主机上的命令查看容器的相关信息

在自己的环境还有在ocp环境上都能使用

| 参数 | 进了哪个 ns               |
|----|-----------------------|
| -n | 网络（必加）                |
| -m | 挂载（看到容器文件系统）          |
| -u | hostname              |
| -i | IPC                   |
| -p | PID（通常别加，否则 ps 只看到自己） |


## 查看网络信息

### 查看容器ID
使用crictl ps 查看容器和Pod的网络信息
```bash
[root@worker2 core]# crictl ps |grep dp-proxy
# 第一列是容器ID
9997611a8c455       d9e4d20e5a7d42c04c422c190fbf3c91f9b98b52303cb19cb86b34c6b7a52756   10 hours ago        Running             dp-proxy                             0                   0f5695e800b0d       dp-proxy-b8468df95-mdzhk                      hytera-controller-trust-ruh
8c70c0c7bd7cc       d9e4d20e5a7d42c04c422c190fbf3c91f9b98b52303cb19cb86b34c6b7a52756   10 hours ago        Running             dp-proxy                             0                   0918ddc790fca       dp-proxy-85d9c7746-vh6g6                      hytera-ysp01-trust-ruh
```

使用容器ID获取容器里面的进程号
```bash
[root@worker2 core]# crictl inspect 9997611a8c455 | grep -i "pid"
    "pid": 530049,
            "type": "pid"
          "pids": {
                "getpid",
                "getppid",
                "pidfd_getfd",
                "pidfd_open",
                "pidfd_send_signal",
                "waitpid",
[root@worker2 core]# 
[root@worker2 core]# crictl inspect -o go-template --template '{{.info.pid}}'  9997611a8c455 
530049
```

```bash
# nsenter -t 530049  -n bash 执行之后，后面所有的操作都是在容器内部网络环境执行的
[root@worker2 core]# nsenter -t 530049  -n bash
[root@worker2 core]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0@if88: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP group default qlen 1000
    link/ether 0a:58:0a:e1:28:0c brd ff:ff:ff:ff:ff:ff link-netns 3f3d1c2e-222a-482c-a9f5-4cb3f280c012
    inet 10.225.40.12/21 brd 10.225.47.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::858:aff:fee1:280c/64 scope link 
       valid_lft forever preferred_lft forever
[root@worker2 core]# ip route
default via 10.225.40.1 dev eth0 
10.225.0.0/16 via 10.225.40.1 dev eth0 
10.225.40.0/21 dev eth0 proto kernel scope link src 10.225.40.12 
100.64.0.0/16 via 10.225.40.1 dev eth0 
169.254.0.5 via 10.225.40.1 dev eth0 
172.31.0.0/16 via 10.225.40.1 dev eth0 
[root@worker2 core]# ip link 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0@if88: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 0a:58:0a:e1:28:0c brd ff:ff:ff:ff:ff:ff link-netns 3f3d1c2e-222a-482c-a9f5-4cb3f280c012
[root@worker2 core]# iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
[root@worker2 core]# ls
 capture_pcap.sh     dc1mcs2.pcap01   dc1mcs2.pcap04  'worker2.z2.ameidc2.com-EmergencyHalf-duplexVoiceCall(E).pcap'
 core@10.161.45.53   dc1mcs2.pcap02   pcap             zone2-worker2-20260717_165551.pcap
 dc1mcs2.pcap00      dc1mcs2.pcap03   tcpdump
[root@worker2 core]# ./tcpdump 
dropped privs to tcpdump
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
^C
0 packets captured
0 packets received by filter
0 packets dropped by kernel
[root@worker2 core]# 
```


## 置身容器内部
使用nsenter能将你从宿主机一步到位移动到完整的容器环境，等价于 `kubectl exec -it app-xxxxxx -- sh`
```bash
# 注意这个时候后面跟的命令，必须是容器里有的命令才能正常使用
nsenter  -t 530084 -n -m -u -i -p sh
```

