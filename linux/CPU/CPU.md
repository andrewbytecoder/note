


## 查找CPU占用内存高的进程

1. 使用 `top -H` 命令查看
2. 按 `f` 打开 ppid查看占用CPU高的进程的父进程PID是多少
```bash
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND    PPID   UID  SUID   GID GROUP      TPGID CGROUPS 
3380256 2030      20   0  771412   6592   1800 R  45.9   0.0 143:51.09 kubectl 3966123  2030  2030  2030 2030          -1 0::/ku+ 
 523668 2030      20   0  772180   9700   1000 R  40.3   0.0  59:36.20 kubectl 3966123  2030  2030  2030 2030          -1 0::/ku+ 
3380462 2030      20   0  771924  10016   2200 R  39.5   0.0  54:52.00 kubectl 3966123  2030  2030  2030 2030          -1 0::/ku+ 
 523669 2030      20   0  772692   9720   1200 R  38.4   0.0  11:28.44 kubectl 3966123  2030  2030  2030 2030          -1 0::/ku+ 
3378302 2030      20   0  772180  10908   2000 R  37.6   0.0  34:29.95 kubectl 3966123  2030  2030  2030 2030          -1 0::/ku+ 
```
3. 使用 `ps -ef | grep 3966123` 找出对应的进程
```bash
2030     3387428 3966123 51 May27 ?        06:38:32 kubectl create token prometheus-k8s -n openshift-monitoring
2030     3530104 3966123 35 May27 ?        04:22:56 kubectl create token prometheus-k8s -n openshift-monitoring
2030     3543906 3966123 57 May27 ?        07:13:00 kubectl create token prometheus-k8s -n openshift-monitoring
2030     3966123 3966118 99 May27 ?        1-10:30:21 /opt/service/kube-opt
core     4145143 4087564  0 01:30 pts/2    00:00:00 grep --color=auto 3966123
```


