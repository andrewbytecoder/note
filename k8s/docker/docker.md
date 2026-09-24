
## network


### network ls
docker 网络类型以及网络列表

```bash
master ✗ $ docker network ls            
NETWORK ID     NAME                                                  DRIVER    SCOPE
57d3b937780d   api7-ee_api7                                          bridge    local
9350c203fc72   bridge                                                bridge    local
731e27faca96   docker_default                                        bridge    local
1342d3e0c1b9   docs_default                                          bridge    local
bcee98080fbd   example_apisix                                        bridge    local
3390f4ec9bb5   grafana_default                                       bridge    local
b4b980d3ed65   gui_default                                           bridge    local
3b2a5c3d7ae5   host                                                  host      local
7172b9cd4555   kind                                                  bridge    local
39c8301d186a   kube-scheduler-simulator_default                      bridge    local
74092f7d0a2b   kube-scheduler-simulator_simulator-internal-network   bridge    local
794f7576a812   neko_default                                          bridge    local
09dc0d2a174e   netviz_default                                        bridge    local
951d6dfc16e9   nexa-network                                          bridge    local
15e9480dbc3a   none                                                  null      local
1fb9e7407b04   pgsql_default                                         bridge    local
bc36eea430d1   prometheus_prometheus_net                             bridge    local
6161882481c7   pyroscope_default                                     bridge    local
92d3554d4903   redis_default                                         bridge    local
9b431b9e5351   samba_default                                         bridge    local
```

### 查看指定网桥的网络信息
`docker network inspect bridge`
```bash
master ✗ $ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "9350c203fc72ff227d6f0fd852e3eb545e28cb1449eb59284801ebf6eb192535",
        "Created": "2026-08-10T10:53:55.83589544+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "3807c5adeae3e9178b4e83739fc8226b07f72ebc7a685db3e0332805569b8d5f": {
                "Name": "buildx_buildkit_mybuilder0",
                "EndpointID": "39dcf9fd3dc07b6fc4676d67340e3284b66fefdb984d1dee90d90766e2747e6d",
                "MacAddress": "da:f1:54:db:8c:1d",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "da34fed598e049a7d0003d48b283eb34a49c7a080d0f6a946b6844dd425d7af2": {
                "Name": "buildx_buildkit_multiarch-builder0",
                "EndpointID": "61e7de3e8c9d39138ea26e0c0608a9291d98d598b5ff97cca31e977246bc30ed",
                "MacAddress": "da:1f:4c:0e:65:ae",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```


## 常用命令

### 镜像操作

#### 将镜像创建一个容器但是不执行用于查看文件和复制文件
```bash
# 创建一个容器，但是不运行
docker create --name skopeo-temp ananace/skopeo:latest
# 将容器中的文件复制出来
docker cp skopeo-temp:/skopeo ./skopeo
# 将容器中所有文件，按照文件列表导出一份列表
docker export skopeo-temp | tar -t > file-list.txt
```





- pdf2zh
```bash
docker pull byaidu/pdf2zh
docker run -d -p 7860:7860 byaidu/pdf2zh
```





### Dockerfile



### 环境注入
像nginx这种服务，通常是使用一个配置 文件来启动，在k8s中不能直接通过环境变量和命令行指定启动的端口或者其他配置参数，这个时候就需要借助envsubst来进行环境变量替换，只需要保证 nginx.conf.template 文件中存在 `${NGINX_{PORT}`}，当执行  `envsubst '$NGINX_PORT'` 之后就会把文件中的变量替换成环境变量。
需要注意的是，如果没有指定 `'$NGINX_PORT'`  `envsubst`会把文件中所有 `$` 开头的变量都进行替换，如果对应的变量不是环境变量就会被替换成空

```dockerfile
# 启动脚本：替换变量并启动 Nginx
CMD envsubst '$NGINX_PORT' < /etc/nginx/nginx.conf.template > /etc/nginx/nginx.conf && \
    echo "✅ Generated nginx.conf:" && cat /etc/nginx/nginx.conf && \
    nginx -g "daemon off;"
```

替换之后，配合k8s的 `env` 配置就能实现环境变量直接注入到 `nginx.conf` 文件


