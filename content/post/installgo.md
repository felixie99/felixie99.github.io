---
title:       "mac配置vscode的go环境"
subtitle:    ""
description: ""
date:        2023-07-25T20:47:02+08:00 
author:      ""
image:       ""
tags:        ["go"]
categories:  ["Tech" ]
katex:       false
---

# 一、安装和配置GO

1. 下载Go的安装包：[安装链接](https://studygolang.com/dl)  

2. 安装完毕后用 go version 验证是否安装成功

3. 使用go env查看go相关的环境变量，主要看GOROOT、GOPATH、GOBIN  
    - GOPATH: GO的工作目录，默认是 ~/go
    - GOROOT: GO的安装目录，默认是/usr/local/go
    - GOBIN: 二进制文件编译目录，这个未配置时应该是空的  

4. 配置环境变量, 使用vim ~/.zshrc, 在末尾添加以下信息
```
# GO
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
export GO111MODULE=on
# GO END`
```  
运行source ~/.zshrc生效配置

5. 使用 go env 查看配置是否已生效

# 二、配置vscode

1. 安装go插件  

2. 使用shift + command + p搜索： >GO: Install/Update Tools全选后确定

# 三、测试go环境

- 在go path下新建一个test 文件夹，然后在里面新建一个test.go文件  

- 写入如下测试代码
```go
package main

import "fmt"

func main() {
	fmt.Println("hello world!")
}
```

- 终端运行命令 go run test.go

# 参考链接
[mac配置vscode和go](http://www.manongjc.com/detail/42-wmzjuaylzzkhcfz.html)  

[Mac配置VsCode+Golang开发环境](https://blog.csdn.net/qyl_0316/article/details/128389748)