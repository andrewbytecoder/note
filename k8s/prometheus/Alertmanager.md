






## 信息查看

### 查看主页面

ip:9093




## 查看alertmanager告警信息

### 查看告警缩略信息

```bash
sh-5.1$ amtool --alertmanager.url=http://localhost:9093 alert query
Alertname                                              Starts At                Summary  State   
hytera-mcs01-trust-dmm_POD_MEMORY_USED_GT_CHECK_44703  2026-10-13 08:35:25 UTC           active  
hytera-mcs01-trust-dmm_POD_MEMORY_USED_GT_CHECK_44703  2026-10-13 08:35:25 UTC           active  
hytera-mcs01-trust-dmm_POD_MEMORY_USED_GT_CHECK_44703  2026-10-13 08:35:25 UTC           active  
hytera-mcs01-trust-dmm_POD_MEMORY_USED_GT_CHECK_44703  2026-10-13 08:35:25 UTC           active  
hytera-mcs01-trust-dmm_POD_MEMORY_USED_GT_CHECK_44703  2026-10-13 08:35:25 UTC           active  
```


### 查看告警详细信息
```bash
sh-5.1$ amtool --alertmanager.url=http://localhost:9093 alert -o extended
Labels                                                                                                                                                                                                                                                                         Annotations                                                                                                                                                              Starts At                Ends At                  Generator URL                                                                                                                                                                               State   
alertname="hytera-mcs01-trust-dmm_POD_MEMORY_USED_GT_CHECK_44703" host_ip="10.161.32.18" namespace="hytera-ysp01-trust-dmm" node="worker5.2z2.ameidc2.com" pod="dis-743623-7659c47bdf-fscgn" pod_ip="10.225.64.69" prometheus="openshift-monitoring/k8s" severity="podcheck"   currentvalue="3.2125" hostIp="10.161.32.18" productId="608369" productName="hytera-mcs01-trust-dmm" serviceName="dis-743623-7659c47bdf-fscgn" threshold="1"              2026-10-13 08:35:25 UTC  2026-10-13 09:22:10 UTC  https://console-openshift-console.apps.2z2.ameidc2.com/monitoring/graph?g0.expr=pod%3Acontainer_memory_usage_percentage%3Asum%7Bnamespace%3D%22hytera-ysp01-trust-dmm%22%7D+%3E+1&g0.tab=1  active
```







[Awesome Prometheus alerts](https://samber.github.io/awesome-prometheus-alerts/)