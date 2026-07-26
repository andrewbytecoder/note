

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

设置代理
```bash
# GOPROXY：模块代理，加速依赖下载（国内必须配置）
go env -w GOPROXY=https://goproxy.cn,direct
```

### 在windows上设置环境变量
在linux上直接将需要设置的环境变量放到可执行文件之前即可实现环境变量的设置，但是在windows上需要使用 `$env` 进行环境变量的指定
- linux
```bash
GODEBUG=gctrace=1 ./app
```
- windows
```bash
$env:GODEBUG="gctrace=1" ./app.exe
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

```bash 
# 模块管理
go mod init github.com/user/project  # 初始化模块
go mod tidy                           # 整理依赖（添加缺失的、删除未使用的）
go mod download                       # 下载依赖到缓存
go mod verify                         # 校验依赖完整性
go mod graph                          # 查看依赖图
go mod vendor                         # 将依赖复制到vendor目录
```


## `go build`


### go build标记
`-x` 输出go build的具体过程
`-n` 查看具体操作而不真正执行
`-v` 查看go build编译的代码包的名称，与  `-a` 标记搭配使用时会非常有用


```bash
# 编译构建
go build -o myapp ./cmd/app     # 编译并指定输出名称
go build -race ./...             # 启用竞态检测编译
```

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

## `go fmt`

```bash
# 代码格式化与静态检查
go fmt ./...                     # 格式化代码
go vet ./...                     # 静态分析，发现常见错误
```


## `go run`

```bash
# 运行go程序过程中，打印详细的gc过程，用于分析内存泄露 
GODEBUG=gctrace=1 go run gColl.go
```
如果在windows上需要使用
```bash
$env:GODEBUG="gctrace=1" go run gColl.go
```

```bash
# 直接运行
go run main.go                   # 编译并立即运行（不生成二进制文件）
go run ./cmd/app                 # 运行指定包
```

## `go get`
### `go get` 标记

`-u` 下载并安装代码包，不论工作区中是否已经存在他们
`-d` 只下载代码包，不安装代码包
`-fix` 下载代码包完成之后，运行一次用于根据当前go语言版本的修正代码的工具，然后再安装代码包
`-t` 同时下载测试需要的代码包
`-insecure` 允许通过非安全的网络协议下载和安装代码包，HTTP就是这样的协议


## `go doc`
```bash
# 文档与信息
go doc fmt.Println               # 查看文档
go doc -all fmt                  # 查看fmt包全部文档
```


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


### 测试
```bash
# 测试
go test ./...                    # 运行所有测试
go test -v ./...                 # 详细输出
go test -race ./...              # 测试时启用竞态检测
go test -cover ./...             # 测试覆盖率
go test -bench=. -benchmem ./... # 基准测试+内存分配统计
```

#### 性能测试

- 执行以下命令之后会在本地生成一个cpu.out文件
```bash
# 指定时间进行性能测试
go test -bench=.  -benchtime=3s -cpuprofile cpu.out
# 直接执行完
go test -bench=.  -cpuprofile cpu.out
```
- 执行 `go tool pprof` 命令查看
```bash
go tool pprof cpu.out
```




















## 工程常用开源库

###  ratelimiter - 限流限频
- 漏桶
[https://github.com/uber-go/ratelimit](https://github.com/uber-go/ratelimit)
- 令牌桶
https://golang.org/x/time/rate

### circuit breaker 熔断器
[https://github.com/sony/gobreaker](https://github.com/sony/gobreaker)

### cache
- interface
[https://github.com/karlseguin/ccache](https://github.com/karlseguin/ccache)
[https://github.com/VictoriaMetrics/fastcache](https://github.com/VictoriaMetrics/fastcache)
[https://github.com/hypermodeinc/ristretto](https://github.com/hypermodeinc/ristretto)

- ringbuffer
[https://github.com/coocood/freecache](https://github.com/coocood/freecache)
[https://github.com/allegro/bigcache](https://github.com/allegro/bigcache)




### json处理

[https://github.com/tidwall/gjson](https://github.com/tidwall/gjson)
[https://github.com/tidwall/sjson](https://github.com/tidwall/sjson)


### 格式转换
- 类型转换，避免使用丑陋的strconv
[https://github.com/spf13/cast](https://github.com/spf13/cast)


## 漏洞扫描

使用以下库可以实现对go应用的CVE漏洞扫描：
```
 go install golang.org/x/vuln/cmd/govulncheck@latest
```

使用示例
```
govulncheck -mode=binary [flags] [binary]
```
