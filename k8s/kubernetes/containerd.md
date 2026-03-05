





## `nsenter`

#### 进入到指定网络空间执行命令

*退回到宿主机的网络命名空间*
```bash
$ nsenter --net=/proc/1/ns/net
```
*列出当前系统中所有网络命名空间*
```bash
[root@k8smaster-ims ~]# lsns -t net
        NS TYPE NPROCS     PID USER                NETNSID NSFS                                                COMMAND
4026531840 net     369       1 root             unassigned                                                     /usr/lib/systemd/sy
4026532297 net       1     778 root             unassigned                                                     /usr/sbin/irqbalanc
4026532409 net       5   18861 65535                     2 /run/netns/cni-7abdfcc6-9aed-11e6-0636-04c71adc65e7 /pause
4026532475 net       2   18672 65535                     1 /run/netns/cni-7eebfd34-23a0-f87b-07ed-ccb84e51c38e /pause
4026532540 net       2   18668 65535                     0 /run/netns/cni-62ed12d3-0ce0-40da-fc82-73a8e831196b /pause
4026532631 net       2   19143 65535                     3 /run/netns/cni-372e9527-3dc3-a2fe-9740-caeb6c43b653 /pause
4026532714 net       3   26022 10000                     4 /run/netns/cni-4a65321f-81ba-4e6c-4064-fc65e6a00245 /pause
```
*使用 `nsenter` 命令进入到指定的命名空间执行命令*
```bash
# 按照NSFS进入到网络命名空间
$ nsenter --net=/run/netns/cni-7abdfcc6-9aed-11e6-0636-04c71adc65e7 ip addr
```


## ctr & crictl
### ctr

#### 加载镜像
```bash
ctr -n k8s.io image import xxx.tar
```



### crictl

#### 查看镜像
```bash
# crictl  images
IMAGE                       TAG                          IMAGE ID            SIZE     <none>     <none>  0577d0df594f9       169MB
10.161.40.40:30003/cgw/ahs  V4.0.01.170.00.51            2e1109f374951       93.1MB
```


#### 查看内存和磁盘占用大小

```bash
[root@k8smaster-40 ~]# crictl stats
CONTAINER           NAME                        CPU %               MEM                 DISK                INODES
012a5b07202cf       redis-exporter              0.00                2.175MB             0B                  8
0296f0b570626       node-exporter               0.00                19.28MB             0B                  15
0553796ea3ab8       wp                          0.59                33.91MB             0B                  30
06972845a9932       llf                         0.33                225MB               30.58MB             20
092a9c629d896       gmkms                       1.71                48.22MB             0B                  16
09da327bed9b3       wpagent                     2.58                67.17MB             0B                  20
0c54f34dafdee       loc                         3.63                59.3MB              0B                  16
1735ce8203b4d       kube-scheduler              0.90                66.42MB             0B                  11
1835eeecd09cc       calico-typha                0.59                43.7MB              0B                  18
19412404e8034       grafana                     0.26                301.3MB             0B                  6
```












