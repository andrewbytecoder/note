



## 网络连通性
### ping 



### nc 

```bash
nc -zv 10.225.30.10 2379
open
```



### `/dev/tcp`
使用echo 直接向 `/dev/tcp` 输出，会创建一个tcp连接

```bash
timeout 3 bash -c 'echo > /dev/tcp/10.226.24.109/14023' && echo "✅ OPEN" || echo "❌ CLOSED/TIMEOUT"
✅ OPEN
```