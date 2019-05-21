---
layout: post
title: "CentOS | Debian" 
date:   2016-05-20 10:35:06
categories:
---

<!-- more -->
### tar

当前目录所有文件移动到上一级目录 `mv * ../`

删除当前目录的所有文件 `rm -rf *`

`tar -c/x` 创建/解压 压缩文档，`tar -f`后接档案名称

`tar -cf o.tar ./`将目录下的文件归档为 o.tar

参数z表示以gzip压缩的 .tar.gz 文件

参数j表示 .tar.bz2 文件


### Putty

全屏设置：`Behaviour ---> Alt+Enter` 所有修改需保存Session配置。
全屏后鼠标移动到左上角单击，可弹出选项菜单，想当于标题栏右键

配置文件 `/etc/ssh/sshd_config`
```
TCPKeepAlive yes
ClientAliveInterval 30
ClientAliveCountMax 60
```
### linux系统用户管理

Unix系统支持多个用户在同一时间内登陆并执行不同的任务、互不影响

实际上，Unix并不识别用户和组的名称，它只识别用户和组对应的 ID 号；分别是用户 ID（User ID，简称 UID）和组 ID（Group ID，简称 GID）

Linux 系统将所有用户名称与 ID 的对应关系都存储在`/etc/passwd`文件，组名称与 ID 的对应关系存储在`/etc/group`文件

创建用户和删除用户
```
adduser 用户名   # 会自动为创建的用户指定主目录、系统shell版本，会在创建时输入用户密码
useradd 用户名   # 需要使用参数选项指定上述基本设置，如果不使用任何参数，则创建的用户无密码、无主目录、没有指定shell版本
userdel 用户名   

