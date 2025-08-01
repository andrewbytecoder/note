

## 交叉编译环境配置

### 在windows上进行交叉编译

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
### 使用vendor管理本地仓库
当开发程序需要在内网运行时，需要使用vendor管理外部仓库，管理过程如下：
```bash
# 下在依赖包
go mod tidy
# 将依赖包更新到vendor这样在内网也能进行打包编译
go mod vendor
```

### 降低依赖包版本
如果部分依赖库的版本需要进行降低，需要在当前库执行 `go get xxx@{version}` 之后在下载依赖包

```bash
# 将 sessions 从 v1.1.4
go get github.com/gorilla/sessions@v1.1.3
# 在更新本地库依赖
go mod tidy 
go mod vendor
```




## `go build`


### go build标记
`-x` 输出go build的具体过程
`-n` 查看具体操作而不真正执行
`-v` 查看go build编译的代码包的名称，与  `-a` 标记搭配使用时会非常有用



打镜像里面的go程序需要启用静态编译
```bash
#!/bin/bash  
  
cur_path=$(pwd)  
  
BuildGoVersion="dpproxy_go build: $(go version)"  
BuildTime="dpproxy_go build: $(date +'%Y-%m-%d %H:%M:%S')"  
DefFlags="-X 'dpproxy_go/config.BuildGoVersion=${BuildGoVersion}' \  
    -X 'dpproxy_go/config.BuildTime=${BuildTime}' "  
  
GoFlags="-a -trimpath"  
  
rm -f dp-proxy  

# 使用静态编译的方式，避免使用动态库导致对环境的依赖
# build -mod=vendor -a -tags netgo

GO111MODULE=on CGO_ENABLED=0 go build -mod=vendor -a -tags netgo -ldflags "${DefFlags}" ${GoFlags} -o dp-proxy  
if [ $? -ne 0 ]; then  
    echo "build failed"  
    exit 1  
fi
```



## `go get`
### `go get` 标记

`-u` 下载并安装代码包，不论工作区中是否已经存在他们
`-d` 只下载代码包，不安装代码包
`-fix` 下载代码包完成之后，运行一次用于根据当前go语言版本的修正代码的工具，然后再安装代码包
`-t` 同时下载测试需要的代码包
`-insecure` 允许通过非安全的网络协议下载和安装代码包，HTTP就是这样的协议





## go test

`go test` 会编译所有 `*_test.go` 文件的包，这些文件可以包含 **功能测试、banchmark测试、模糊测试。** 所有以 `_` 和 `.` 开头的文件都会被忽略，包括 `_test.go`

在 `go test` 前会运行 `go vet` 测试文件签名问题，如果在 `go vert` 过程中发现问题，将停止运行 `go test`。

**`go test` 的两种模式：**

- `go test` 编译运行当前文件测试文件。侧重模式下不 `caching`
- `go test .` ，`go test math`，`go test ./...`

### 其他参数
- `-benchtime t`：指定测试时间，比如 1s 指定运行 1s
- `-count n` : 指定运行多少次用例。如果 `-cpu` 指定了 cpu 数量，运行次数将会是 `[n * cpu]`
- `-cover`：输出代码测试覆盖率
- `-json` ：以json方式输出测试报告
- `-failfast` ：一个测试用例失败后不去测试其他的测试用例
- `-run regexp`：指定运行测试的规则
- `-cpu 1,2,4`：指定 cpu 数量，会基于一个 cpu 跑一起，基于 2 个 cpu 跑一次，基于 4 个跑一次。对于benchmarks和模糊测试有意义。


