





- 搜集pprof的所有信息-node-exporter
```bash
for p in goroutine block mutex profile; do
  curl -O localhost:9100/debug/pprof/$p
done
```