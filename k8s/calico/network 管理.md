



## ipvsadm 


### 查看ClusterIP -> Pod的流量统计 ipvsadm -Ln --stats
```bash
[root@k8smaster-ims ~]# ipvsadm -Ln --stats
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -> RemoteAddress:Port
TCP  10.96.0.1:443                   97081 49620559 49636689   10134M   49228M
  -> 10.168.8.107:6443               97081 49620559 49636689   10134M   49228M
TCP  10.96.0.10:53                       0        0        0        0        0
  -> 10.250.40.0:53                      0        0        0        0        0
  -> 10.250.40.3:53                      0        0        0        0        0
TCP  10.96.0.10:9153                     0        0        0        0        0
```

### 查看ClusterIP -> Pod 的实时速率 ipvsadm -Ln --rate
```bash
[root@k8smaster-ims ~]# ipvsadm -Ln --rate
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port                 CPS    InPPS   OutPPS    InBPS   OutBPS
  -> RemoteAddress:Port
TCP  10.96.0.1:443                       0       11       11     1240     8828
  -> 10.168.8.107:6443                   0       11       11     1240     8828
TCP  10.96.0.10:53                       0        0        0        0        0
```



### 查看连接超时 ipvsadm -L --timeout
```bash
[root@k8smaster-ims ~]# ipvsadm -L --timeout
Timeout (tcp tcpfin udp): 900 120 300
```

### 查看当前IPVS连接跟踪表 ipvsadm -Ln -c
用来查看连接打到了哪个Pod上
```bash
[root@k8smaster-ims ~]# ipvsadm -Lnc
IPVS connection entries
pro expire state       source             virtual            destination
TCP 00:23  TIME_WAIT   10.250.40.8:33086  10.96.1.169:443    10.250.40.5:8443
UDP 00:43  UDP         10.250.40.8:47011  10.96.0.10:53      10.250.40.0:53
UDP 03:13  UDP         10.250.40.8:56858  10.96.0.10:53      10.250.40.3:53
UDP 02:01  UDP         10.250.40.39:58860 10.96.0.10:53      10.250.40.3:53
UDP 03:46  UDP         10.250.40.38:44032 10.96.0.10:53      10.250.40.0:53
UDP 02:01  UDP         10.250.40.39:54657 10.96.0.10:53      10.250.40.0:53
```



## nft 

### `nft list ruleset`
使用nft命令查看calico 使用felix在一个node上建立的整张网络管理表格




