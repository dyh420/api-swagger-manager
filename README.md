# Authority for services

### 维护必知

#### requirement

Golang version >= 1.12.0


#### 包管理

*目前还在试验性阶段，不确定为最佳实践，其他项目谨慎参考*


##### 准备阶段

1. 确定开发环境安装 Golang >= 1.12.0 版本
2. 检查 `GO111MODULE` 环境变量是否开启
3. git >= 2.21.0

```shell
# 没有设置的情况下，返回为空
$ echo $GO111MODULE

# 开启该功能
$ export GO111MODULE=on
$ echo $GO111MODULE
on
# 已经开启该功能
```

#### 添加自己的依赖

1. 确保当前下面的目录下已经存在 **go.mod** 文件。如果没有，执行 `go mod init` 自动生成
2. 完成业务开发与维护。执行如下操作。

```shell
# 分析代码并找出依赖，下载
$ go mod tidy
go: finding github.com/jinzhu/gorm/dialects/mysql latest
...

# 将依赖合并复制到项目 vendor 目录下。初次执行到时候，会比较慢。
$ go mod vendor
```

* 执行下载的时候，部分依赖需要使用到 golang.org 的库，由于特殊政策，确保开发环境是可以科学上网 *


## 编译

```shell
#提示：执行该编译命令前请确保工程路径是 $GOPATH/src/git-pd.megvii-inc.com/Data-Core/Infrastructure/apidoc-backend
$ docker run -it --rm --name go -v `pwd`:/go/src/git-pd.megvii-inc.com/Data-Core/Infrastructure/apidoc-backend -w=/go/src/git-pd.megvii-inc.com/Data-Core/Infrastructure/apidoc-backend/cmd/apidoc-backend-cli golang:1.12 go build -v -ldflags '-s -w'
```
## 启动
### 参数说明：
1. config 参数可在启动命令中设置如方法1， 也可在apollo中设置如方法2
2. 如果启动命令中设置了config 参数，则会优先使用本地config文件中的配置
3. 如果启动命令未设置config参数，则从apollo平台拉取配置，这需要在系统环境变量中配置apollo接口信息，如下

```shell
#---------apollo agent setting------------------------
$ export RUN_ENV=local
#-----------------------------------------------------
```

### 启动方法

```shell
#1. 启动 server 方法1: 
$ ./apidoc-backend-cli --config config.yaml server    
#2. 启动 server 方法2(免编译): 
$ go run apidoc-backend/cmd/main.go server
```

## 测试
### 服务端测试

```shell
#业务代码整体测试
$ cd internal/server
$ go test
```

### 客户端测试

```shell
#测试获取 config 
$ ./apidoc-backend-cli --server 127.0.0.1:9988 config get -f 17016291999117712
```

## 配置文件
config.yaml:
```
name: service_name
listen: 9988
EnableKubernetesAuth: false
db_type: mysql
db_connect_string: username:password@(host)/database?charset=utf8&parseTime=True
enable_auto_migrate: true
oss_connect_string: oss://key:secret@bucket.endpoint/prefix
uuid_url_string: https://api.zzcrowd.com/identity/v2/uid
log_level: debug
```

## 常见问题
一. go get *** 或 go mod tidy 发生类似unknown revision问题
1. 先确认go version 和 git version 达到文首要求的版本
2. 如果是go get git-pd.megvii-inc.com/Data-Common/golib-x 出错，确认在浏览器中是否有权限访问该项目 
3. 修改环境变量：export GIT_TERMINAL_PROMPT=1
4. 这可能使得在go get 时输入git 的账户和密码，如果在macOS中开发， 可以使用系统自带的账户密码管理功能，
   如果在Linux 中开发， 通过缓存账户密码解决， 例如：

   ```shell
   $ git config --global credential.helper store
   #或者
   $ vim ~/.gitconfig
     [credential]
         helper = store 
   ```  
