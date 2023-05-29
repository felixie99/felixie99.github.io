---
title:       "代理服务器"
subtitle:    ""
description: ""
date:        2023-05-29T14:31:23+08:00 
author:      ""
image:       ""
tags:        ["linux"]
categories:  ["Tech" ]
katex:       false
---

# SSH命令的三种代理功能 (-L/-R/-D)  
[搬运自知乎答主](https://zhuanlan.zhihu.com/p/57630633)
ssh命令的三种代理功能:
- 正向代理(-L)：相当于iptable的port forwarding
- 反向代理(-R)：相当于frp或者ngrok
- socks5代理(-D)：相当于 ss/ssr

### 正向代理：  

即在本地启动端口，把本地端口数据转发到远端  

用法1:远程端口映射到其他机器  
HostB 上启动一个 PortB 端口，映射到 HostC:PortC 上，在 HostB 上运行：
``` 
HostB$ ssh -L 0.0.0.0:PortB:HostC:PortC user@HostC
```  
这时访问 HostB:PortB 相当于访问 HostC:PortC（和 iptable 的 port-forwarding 类似）。

用法2：本地端口通过跳板映射到其他机器

HostA 上启动一个 PortA 端口，通过 HostB 转发到 HostC:PortC上，在 HostA 上运行：
```
HostA$ ssh -L 0.0.0.0:PortA:HostC:PortC  user@HostB
```
这时访问 HostA:PortA 相当于访问 HostC:PortC。

两种用法的区别是，第一种用法本地到跳板机 HostB 的数据是明文的，而第二种用法一般本地就是 HostA，访问本地的 PortA，数据被 ssh 加密传输给 HostB 又转发给 HostC:PortC。  

### 反向代理  

所谓“反向代理”就是让远端启动端口，把远端端口数据转发到本地。

HostA 将自己可以访问的 HostB:PortB 暴露给外网服务器 HostC:PortC，在 HostA 上运行：
```
HostA$ ssh -R HostC:PortC:HostB:PortB  user@HostC
```
那么链接 HostC:PortC 就相当于链接 HostB:PortB。使用时需修改 HostC 的 /etc/ssh/sshd_config，添加：
```
GatewayPorts yes
```
相当于内网穿透，比如 HostA 和 HostB 是同一个内网下的两台可以互相访问的机器，HostC是外网跳板机，HostC不能访问 HostA，但是 HostA 可以访问 HostC。

那么通过在内网 HostA 上运行 ssh -R 告诉 HostC，创建 PortC 端口监听，把该端口所有数据转发给我（HostA），我会再转发给同一个内网下的 HostB:PortB。

同内网下的 HostA/HostB 也可以是同一台机器，换句话说就是内网 HostA 把自己可以访问的端口暴露给了外网 HostC。

按照前文《韦易笑：内网穿透：在公网访问你家的 NAS》中，相当于再 HostA 上启动了 frpc，而再 HostC 上启动了 frps。