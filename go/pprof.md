


## `go tool pprof`

### 收集堆内存数据
```bash
go tool pprof http://localhost:6060/debug/pprof/heap
```

#### 内存分析命令
- `top -cum` 查看累计内存分配
- `inuse_space` 正在使用的内存
- `alloc_space` 历史内存分配


### 性能分析(生成火焰图)
```bash
go tool pprof -http:808 http://localhost:6060/debug/pprof/profile
```


### 性能对比分析
- 内存泄露
- 协程泄露
- 性能优化前后对比
```bash
go tool pprof -base old.pprof new.pprof
```


### 常用分析方式
- 生产环境： 谨慎使用CPU Profile，会影响性能
- 长期运行服务：定期采集内存profile对比
- 基准测试：结合go test -bench和pprof
- 容器环境：确保pprof端口可以访问

> 经验法则：先通过top命令找到热点，再通过list深入分析，最后使用web可视化确认



- 搜集pprof的所有信息-node-exporter
```bash
for p in goroutine block mutex profile; do
  curl -O localhost:9100/debug/pprof/$p
done
```







https://blog.wolfogre.com/posts/go-ppof-practice/

https://farmerchillax.github.io/2023/07/04/Go%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90%E5%B7%A5%E5%85%B7/