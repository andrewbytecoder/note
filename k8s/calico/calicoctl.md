
##  常用命令

### 查看Pod网络信息
```bash
[root@k8smaster-1 ~]# calicoctl get workloadendpoints -A -o wide 
NAMESPACE       NAME                                                                 WORKLOAD                                   NODE          NETWORKS            INTERFACE         PROFILES                                                      NATS   
base-services   k8smaster--1-k8s-alertmanager--main--0-eth0                          alertmanager-main-0                        k8smaster-1   100.70.244.235/32   cali3dbf08c6d60   kns.base-services,ksa.base-services.alertmanager-main                
base-services   k8smaster--1-k8s-alertmanager--main--1-eth0                          alertmanager-main-1                        k8smaster-1   100.70.244.209/32   cali5459fa44f4f   kns.base-services,ksa.base-services.alertmanager-main                
base-services   k8smaster--1-k8s-alertmanager--main--2-eth0                          alertmanager-main-2                        k8smaster-1   100.70.244.212/32   cali09fe7018ef7   kns.base-services,ksa.base-services.alertmanager-main                
base-services   k8smaster--1-k8s-grafana--76bc9656b8--jmc5d-eth0                     grafana-76bc9656b8-jmc5d                   k8smaster-1   100.70.244.203/32   cali4419acc64f7   kns.base-services,ksa.base-services.grafana                          
base-services   k8smaster--1-k8s-kube--state--metrics--65c9499785--l778b-eth0        kube-state-metrics-65c9499785-l778b        k8smaster-1   100.70.244.201/32   cali6fd8b36a4dd   kns.base-services,ksa.base-services.kube-state-metrics               
base-services   k8smaster--1-k8s-prometheus--adapter--56c65ffd9b--lvwcz-eth0         prometheus-adapter-56c65ffd9b-lvwcz        k8smaster-1   100.70.244.228/32   cali4535aa83780   kns.base-services,ksa.base-services.prometheus-adapter               
base-services   k8smaster--1-k8s-prometheus--k8s--0-eth0                             prometheus-k8s-0                           k8smaster-1   100.70.244.230/32   caliee699845661   kns.base-services,ksa.base-services.prometheus-k8s                   
base-services   k8smaster--1-k8s-prometheus--operator--656c46b8d8--jjkqv-eth0        prometheus-operator-656c46b8d8-jjkqv       k8smaster-1   100.70.244.225/32   caliab880c31cf9   kns.base-services,ksa.base-services.prometheus-operator              
calico-system   k8smaster--1-k8s-calico--kube--controllers--7c8b4f9f84--fts68-eth0   calico-kube-controllers-7c8b4f9f84-fts68   k8smaster-1   100.70.244.218/32   cali08c24e5fb05   kns.calico-system,ksa.calico-system.calico-kube-controllers          
calico-system   k8smaster--1-k8s-csi--node--driver--ph22g-eth0                       csi-node-driver-ph22g                      k8smaster-1   100.70.244.193/32   calic2eeda9bb38   kns.calico-system,ksa.calico-system.default                          
kube-system     k8smaster--1-k8s-coredns--75989b4c59--fjv84-eth0                     coredns-75989b4c59-fjv84                   k8smaster-1   100.70.244.234/32   cali1e6c2b85682   kns.kube-system,ksa.kube-system.coredns                              
kube-system     k8smaster--1-k8s-coredns--75989b4c59--ldllj-eth0                     coredns-75989b4c59-ldllj                   k8smaster-1   100.70.244.232/32   cali57c605ce2cc   kns.kube-system,ksa.kube-system.coredns                              

[root@k8smaster-1 ~]# 
```




## ipam信息

### ipam信息查看
```bash
[root@k8smaster-1 ~]# calicoctl ipam show
+----------+---------------+------------+------------+-------------------+
| GROUPING |     CIDR      | IPS TOTAL  | IPS IN USE |     IPS FREE      |
+----------+---------------+------------+------------+-------------------+
| IP Pool  | 100.64.0.0/10 | 4.1943e+06 | 13 (0%)    | 4.1943e+06 (100%) |
+----------+---------------+------------+------------+-------------------+

[root@k8smaster-1 ~]# calicoctl ipam show --show-blocks
+----------+-------------------+------------+------------+-------------------+
| GROUPING |       CIDR        | IPS TOTAL  | IPS IN USE |     IPS FREE      |
+----------+-------------------+------------+------------+-------------------+
| IP Pool  | 100.64.0.0/10     | 4.1943e+06 | 13 (0%)    | 4.1943e+06 (100%) |
| Block    | 100.70.244.192/26 |         64 | 13 (20%)   | 51 (80%)          |
+----------+-------------------+------------+------------+-------------------+
```



### 查看对应的IP是谁在使用

```bash
[root@k8smaster-1 ~]# calicoctl ipam show --ip=100.70.244.212
IP 100.70.244.212 is in use
Attributes:
  node: k8smaster-1
  pod: alertmanager-main-2
  timestamp: 2026-09-20 14:44:27.398090919 +0000 UTC
  namespace: base-services
```




