
## 使用到的一些资源

### `NetworkPolicy`


![](attachments/Pasted%20image%2020250225202348.png)

- 通过 `kubectl client` 创建 network policy 资源
- calico 的 policy -controller 监听 network policy 资源，获取后写入calico的etcd数据库
- node 上的calico-felix 从etcd数据库获取policy资源，调用iptables做相应配置

### `Role`(`ClusterRole`) && `RoleBinding`(`ClusterRoleBinding`)

该Role含有的权限，通过`RoleBinding`之后就会赋予对应的User, Group, Service Account相应的权限

![](attachments/Pasted%20image%2020250225210010.png)



## node-exporter

node-exporter中有两个容器：
1. kube-rabc-proxy : 鉴权，代理
2. node-exporter: 数据采集


在kube-prometheus中存在很多这种经典的设计，因为需要进行鉴权，两个容器如果127.0.0.1和podIP都进行监听的话，那么必然需要两个IP，那么只需要将node-export只监听127.0.0.1:9100然后kube-rabc-proxy监听 podIP:9100，然后再使用kube-rabc-proxy代理node-export的9100，经过这样的操作之后整个pod只需要对外提供一个9100端口就行。

![](attachments/Pasted%20image%2020250423170607.png)

详见：[github issue单2146](https://github.com/prometheus-operator/prometheus-operator/pull/2146)



## 抓取目标

在kube-prometheus中通过 `ServiceMonitor` 来发现目标，如果哪个service想要被prometheus抓取数据，只需要配置一个 `ServiceMonitor`对象即可，配置之后在 `prometheus` 的 `Targets` 页面就会发现对应的服务，并按照执行的服务端口抓取数据。

```yaml
apiVersion: monitoring.coreos.com/v1  
kind: ServiceMonitor  
metadata:  
  labels:  
    app.kubernetes.io/component: grafana  
    app.kubernetes.io/name: grafana  
    app.kubernetes.io/part-of: kube-prometheus  
    app.kubernetes.io/version: 11.6.0  
  name: grafana  
  namespace: monitoring  
spec:  
  endpoints:  
  - interval: 15s  
    port: http  
  selector:  
    matchLabels:  
      app.kubernetes.io/name: grafana
```


如果是 pod资源可以使用如下配置
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
spec:
  scrapeClass: istio-mtls
  podMetricsEndpoints:
  - port: http
    path: /metrics
```







## ctr

kube-prometheus镜像加载使用命令 `ctr -n k8s.io image import sipaas_V5.0.02.030.36.tar` 需要指定命名空间为k8s.io







