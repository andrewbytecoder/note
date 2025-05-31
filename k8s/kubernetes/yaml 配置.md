

## image相关

### **1. `imagePullPolicy` 的三种模式**

```yaml
containers:
- name: my-container
  image: 10.161.40.240:30003/my/lab:latest
  imagePullPolicy: IfNotPresent          
  volumeMounts:
	- name: config-volume
	  mountPath: /etc/nginx
```

Kubernetes 支持以下三种 `imagePullPolicy` 模式：

#### (1) `Always`
- **默认行为**：如果未指定 `imagePullPolicy`，且镜像标签为 `:latest`，则默认使用此模式。
- **行为**：
  - 每次启动 Pod 时，Kubernetes 都会尝试从远程镜像仓库拉取最新的镜像。
  - 如果远程镜像仓库不可用或镜像不存在，Pod 启动会失败。
- **适用场景**：
  - 希望确保始终使用最新版本的镜像（例如开发环境）。
  - 镜像经常更新，且需要及时获取最新版本。

#### (2) `IfNotPresent`
- **行为**：
  - 如果本地已经存在镜像，则直接使用本地镜像，不会尝试从远程拉取。
  - 如果本地不存在镜像，则尝试从远程镜像仓库拉取。
- **适用场景**：
  - 希望优先使用本地镜像，但允许在本地镜像不存在时拉取远程镜像。
  - 常用于离线环境或希望减少网络依赖的场景。

#### (3) `Never`
- **行为**：
  - 始终使用本地镜像，不会尝试从远程镜像仓库拉取。
  - 如果本地不存在镜像，Pod 启动会失败。
- **适用场景**：
  - 完全离线环境，无法访问远程镜像仓库。
  - 确保只使用本地镜像（例如安全要求或测试环境）。

---

## 生命周期管理

```yaml
containers:
- name: my-container
  image: 10.161.40.240:30003/my/lab:latest
  imagePullPolicy: IfNotPresent          
  lifecycle:
	postStart: # 容器启动后立即执行一个指定的操作，执行失败会导致容器启动失败
		exec:
			command: ["/bin/sh", "-c", "echo hello from the postStart handler"]
	preStop: # 容器被杀死之前，收到 SIGKILL信号时，只有该命令执行成功才允许容器被杀死
		exec:
			command: ["/usr/sbin/nginx", "-s", "quit"]
```