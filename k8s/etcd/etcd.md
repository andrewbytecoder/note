








- 查询当前节点状态
```bash
./etcdctl --endpoints=$ENDPOINTS endpoint status --write-out=table
# 使用示例
 etcdctl  --endpoints=10.161.48.46:12256,10.161.48.45:12256 endpoint status --write-out=table
```



配置信息

```bash
# 配置信息目录
/etc/etcd/etcd.conf

ETCDCTL_API=3 etcdctl --endpoints=https://10.161.40.73:2379,https://10.161.40.72:2379,https://10.161.42.189:2379 --cacert=/etc/k8s/cert/ca.pem --cert=/etc/k8s/cert/etcd.pem --key=/etc/k8s/cert/etcd-key.pem endpoint status -w table

```




在关于集群的可用性（ availability） 这一点上， etcd 认为一致性比可用性更加重要。这意味着 etcd 在出现脑裂的情况时，会停止为集群提供更新能力，来保证存储数据的一致性