



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




