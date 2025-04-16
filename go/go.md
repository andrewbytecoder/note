

### 交叉编译环境配置

编译linux上程序设置
```bash
set GOARCH=amd64
go env -w GOARCH=amd64
set GOOS=linux
go env -w GOOS=linux
```
切换回windows

```bash
go env -w GOARCH=amd64
go env -w GOOS=windows
```


## `go mod`
### 清理依赖缓存

如果某些库已经删除，但是其他库存在依赖关系，可以使用一下命令清理mod缓存
```bash
go clean -modcache 
go mod tidy
```



