



## 镜像


### 查看镜像

```bash
ctr -n k8s.io images list 
```


### 挂载镜像
将镜像展看到本地 `/mnt`  目录
```bash
ctr -n k8s.io images mount docker.io/nginx/nginx-gateway-fabric:v1.26.7 /mnt
```


