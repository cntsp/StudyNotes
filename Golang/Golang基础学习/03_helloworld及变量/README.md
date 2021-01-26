# 第一个Go程序及变量

## Hello World

现在我们来创建第一个Go项目--`hello`。在我们的`GOPATH`下 `src`目录中创建hello目录。

![image-20210126220258950](images/image-20210126220258950.png)

![image-20210126220355085](images/image-20210126220355085.png)

在该目录中创建一个`main.go`文件：

```go
package main  // 声明 main 包， 表明当前是一个可执行程序

import "fmt"  // 导入内置 fmt 包

func main() {  // main函数，是程序执行的入口
    fmt.Println("Hello World!")   // 在终端打印 Hello World!
}
```



## go module(重点)

Go语言的依赖管理随着版本的更迭正逐渐完善起来。

### 为什么需要依赖管理？

最早的时候，Go所依赖的所有的第三方库都放在GOPATH这个目录下面。这就导致了同一个库只能保存一个版本的代码。如果不同的项目依赖同一个第三方的库的不同版本，应该怎么解决？

### godep

Go语言从v1.5开始开始引入`vendor`模式，如果项目目录下有vendor目录，那么go工具链就会优先使用`vendor`内的包进行编译、测试等。

`godep`是一个通过vender模式实现的Go语言的第三方依赖管理工具，类似的还有社区维护准官方包管理工具`dep`。

### 安装

```go
go get github.com/tools/godep
```

### 基本命令

安装好godep之后，在终端输入`godep`查看支持的所有命令。

```go
godep save     将依赖项输出并复制到Godeps.json文件中
godep go       使用保存的依赖项运行go工具
godep get      下载并安装具有指定依赖项的包
godep path     打印依赖的GOPATH路径
godep restore  在GOPATH中拉取依赖的版本
godep update   更新选定的包或go版本
godep diff     显示当前和以前保存的依赖项集之间的差异
godep version  查看版本信息
```

使用`godep help [command]`可以看看具体命令的帮助信息。

### 使用godep

在项目目录下执行`godep save`命令，会在当前项目中创建`Godeps`和`vender`两个文件夹。

其中`Godeps`文件夹下有一个`Godeps.json`的文件，里面记录了项目所依赖的包信息。 `vender`文件夹下是项目依赖的包的源代码文件。



### vender机制

Go1.5版本之后开始支持，能够控制Go语言程序编译时依赖包搜索路径的优先级。

例如查找项目的某个依赖包，首先会在项目根目录下的`vender`文件夹中查找，如果没有找到就会去`$GOAPTH/src`目录下查找。



### godep开发流程

1. 保证程序能够正常编译
2. 执行`godep save`保存当前项目的所有第三方依赖的版本信息和代码
3. 提交Godeps目录和vender目录到代码库。
4. 如果要更新依赖的版本，可以直接修改`Godeps.json`文件中的对应项



### go module上场

`go module`是Go1.11版本之后官方推出的版本管理工具，并且从Go1.13版本开始，`go module`将是Go语言默认的依赖管理工具。

#### GO111MODULE

要启用`go module`支持首先要设置环境变量`GO111MODULE`，通过它可以开启或关闭模块支持，它有三个可选值：`off`、`on`、`auto`，默认值是`auto`。

1. `GO111MODULE=off`禁用模块支持，编译时会从`GOPATH`和`vendor`文件夹中查找包。
2. `GO111MODULE=on`启用模块支持，编译时会忽略`GOPATH`和`vendor`文件夹，只根据 `go.mod`下载依赖。
3. `GO111MODULE=auto`，当项目在`$GOPATH/src`外且项目根目录有`go.mod`文件时，开启模块支持。

简单来说，设置`GO111MODULE=on`之后就可以使用`go module`了，以后就没有必要在GOPATH中创建项目了，并且还能够很好的管理项目依赖的第三方包信息。

使用 go module 管理依赖后会在项目根目录下生成两个文件`go.mod`和`go.sum`。

### GOPROXY

Go1.11之后设置GOPROXY命令为：

```go
export GOPROXY=https://goproxy.cn
```

Go1.13之后`GOPROXY`默认值为`https://proxy.golang.org`，在国内是无法访问的，所以十分建议大家设置GOPROXY，这里我推荐使用[goproxy.cn](https://studygolang.com/topics/10014)。

```go
go env -w GOPROXY=https://goproxy.cn,direct
```

### go mod命令

常用的`go mod`命令如下：

```go
go mod download    下载依赖的module到本地cache（默认为$GOPATH/pkg/mod目录）
go mod edit        编辑go.mod文件
go mod graph       打印模块依赖图
go mod init        初始化当前文件夹, 创建go.mod文件
go mod tidy        增加缺少的module，删除无用的module
go mod vendor      将依赖复制到vendor下
go mod verify      校验依赖
go mod why         解释为什么需要依赖
```

### go.mod文件

go.mod文件记录了项目所有的依赖信息，其结构大致如下：

```go
module cmcc/ifct/dialogue/hxb-backend

go 1.15

replace cmcc/ifct/common/go => ../../common/go

replace cmcc/ifct/common/proto/gen/go/common => ../../common/proto/gen/go/common

replace cmcc/ifct/common/proto/gen/go/hxb => ../../common/proto/gen/go/hxb

replace cmcc/ifct/common/proto/gen/go/moon => ../../common/proto/gen/go/moon

replace cmcc/ifct/common/proto/gen/go/chatbot => ../../common/proto/gen/go/chatbot

require (
	cmcc/ifct/common/go v0.0.0-00010101000000-000000000000
	cmcc/ifct/common/proto/gen/go/chatbot v0.0.0-00010101000000-000000000000 // indirect
	cmcc/ifct/common/proto/gen/go/common v0.0.0-00010101000000-000000000000
	cmcc/ifct/common/proto/gen/go/hxb v0.0.0-00010101000000-000000000000
	cmcc/ifct/common/proto/gen/go/moon v0.0.0-00010101000000-000000000000
	github.com/disintegration/imageorient v0.0.0-20180920195336-8147d86e83ec
	github.com/go-basic/uuid v1.0.0
	github.com/golang/glog v0.0.0-20160126235308-23def4e6c14b
	github.com/gorilla/mux v1.8.0
	github.com/jinzhu/gorm v1.9.16
	github.com/jinzhu/now v1.1.1 // indirect
	github.com/rs/cors v1.7.0
	github.com/stretchr/testify v1.6.1
	google.golang.org/grpc v1.34.1
)
```

其中，

- `module`用来定义包名
- `require`用来定义依赖包及版本
- `indirect`表示间接引用



### 依赖的版本

go mod支持语义化版本号，比如`go get foo@v1.2.3`，也可以跟git的分支或tag，比如`go get foo@master`，当然也可以跟git提交哈希，比如`go get foo@e3702bed2`。关于依赖的版本支持以下几种格式：

```go
gopkg.in/tomb.v1 v1.0.0-20141024135613-dd632973f1e7
gopkg.in/vmihailenco/msgpack.v2 v2.9.1
gopkg.in/yaml.v2 <=v2.2.1
github.com/tatsushid/go-fastping v0.0.0-20160109021039-d7bb493dee3e
latest
```

### replace

在国内访问golang.org/x的各个包都需要翻墙，你可以在go.mod中使用replace替换成github上对应的库。

```gp
replace (
	golang.org/x/crypto v0.0.0-20180820150726-614d502a4dac => github.com/golang/crypto v0.0.0-20180820150726-614d502a4dac
	golang.org/x/net v0.0.0-20180821023952-922f4815f713 => github.com/golang/net v0.0.0-20180826012351-8a410e7b638d
	golang.org/x/text v0.3.0 => github.com/golang/text v0.3.0
)
```

### go get

在项目中执行`go get`命令可以下载依赖包，并且还可以指定下载的版本。

1. 运行`go get -u`将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号)
2. 运行`go get -u=patch`将会升级到最新的修订版本
3. 运行`go get package@version`将会升级到指定的版本号version

如果下载所有依赖可以使用`go mod download`命令。



### 整理依赖

我们在代码中删除依赖代码后，相关的依赖库并不会在`go.mod`文件中自动移除。这种情况下我们可以使用`go mod tidy`命令更新`go.mod`中的依赖关系。



### go mod edit

#### 格式化

因为我们可以手动修改go.mod文件，所以有些时候需要格式化该文件。Go提供了一下命令：

```bash
go mod edit -fmt
```

### 添加依赖项

```bash
go mod edit -require=golang.org/x/text
```

### 移除依赖项

如果只是想修改`go.mod`文件中的内容，那么可以运行`go mod edit -droprequire=package path`，比如要在`go.mod`中移除`golang.org/x/text`包，可以使用如下命令：

```bash
go mod edit -droprequire=golang.org/x/text
```

关于`go mod edit`的更多用法可以通过`go help mod edit`查看。



### 在项目中使用 go module

### 既有项目

如果需要对一个已经存在的项目启用`go module`，可以按照以下步骤操作：

1. 在项目目录下执行`go mod init`，生成一个`go.mod`文件。
2. 执行`go get`，查找并记录当前项目的依赖，同时生成一个`go.sum`记录每个依赖库的版本和哈希值。

### 新项目

对于一个新创建的项目，我们可以在项目文件夹下按照以下步骤操作：

1. 执行`go mod init 项目名`命令，在当前项目文件夹下创建一个`go.mod`文件。
2. 手动编辑`go.mod`中的require依赖项或执行`go get`自动发现、维护依赖。



## go build

使用`go build`

1. 在项目目录下执行`go build`
2. 在其他路径下执行`go build`，需要在后面加上项目的路径（项目路径从`GOPATH/src`后开始写起,编译之后的可执行文件就在当前目录中）
3. 如果不想使用默认的二进制可执行文件名（默认情况下可执行文件名为项目名.exe），可以使用`-o` 参数指定生成的可执行文件名

![image-20210126230041217](images/image-20210126230041217.png)

![image-20210126230844108](images/image-20210126230844108.png)



## go run

向执行脚本文件一样执行Go代码



## go install

`go install` 分两步：

1. 先编译得到一个可执行文件
2. 将可执行文件拷贝到`GOPATH/bin`

`go install`表示安装的意思：它先编译源代码得到可执行文件，然后将可执行文件移动到`GOPATH`的bin目录下。因为我们的环境变量中配置了`GOPATH`下的bin目录，所以我们就可以在任意地方直接执行可执行文件了。



## 跨平台编译

默认我们 `go build` 的可执行文件都是当前操作系统可执行的文件。

如果我想在windows下编译一个`linux`下可执行文件，那需要怎么做呢?

只需要指定目标操作系统的平台和处理器架构即可：

```go
SET CGO_ENABLED=0 // 禁用CGO
SET GOOS=linux // 目标平台是linux
SET GOARCH=amd64  //目标处理器架构是amd64
```

然后再执行`go build`命令，得到的就是能够在Linux平台运行的可执行文件了。

Mac 下编译 Linux 和 Windows 平台64位可执行程序：

```go
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

Linux  下编译 Mac 和 Windows 平台64位可执行程序：

```go
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build
```

Windows 下编译Mac平台64位可执行程序：

```go
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build
```



## 变量

### 变量声明

### 变量赋值

### 常量和iota

### 整型

### 浮点型负数和布尔值

 