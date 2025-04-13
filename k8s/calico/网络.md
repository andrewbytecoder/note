
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


### 查看对应Pod属于哪个网卡

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