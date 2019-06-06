---
layout: post
title: "CentOS | Debian" 
date:   2016-05-20 10:35:06
categories:
---

<!-- more -->

### 修改主机名,重启生效
`hostnamectl set-hostname NAME`

### 旧版本的仓库被归档

打开`/etc/apt/sources.list`配置文件,编辑源地址

发行文档 https://wiki.debian.org/DebianReleases

### 查询指定包的详情
```
apt-cache show package
```

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
### linux系统用户管理与组管理

Unix系统支持多个用户在同一时间内登陆并执行不同的任务、互不影响

实际上，Unix并不识别用户和组的名称，它只识别用户和组对应的 ID 号；分别是用户 ID（User ID，简称 UID）和组 ID（Group ID，简称 GID）

Linux 系统将所有用户名称与 ID 的对应关系都存储在`/etc/passwd`文件，组名称与 ID 的对应关系存储在`/etc/group`文件

`/etc/passwd`文件中的内容非常规律，每行记录对应一个用户,不仅如此，每行用户信息都以":"作为分隔符，划分为7个字段

```
用户名：密码：UID（用户ID）：初始组的GID：描述性信息：主目录：默认Shell
```

`/ect/group`文件是用户组配置文件，`etc/passwd`文件中每行用户信息的第四个字段记录的是用户的初始组ID,而用户组的所有信息都存放在`/ect/group`文件中，此文件每行记录对应一个用户组，以":"划分为4个字段

```
组名：密码：GID：该用户组中的（附加）用户列表
```


创建用户和删除用户
```
adduser 用户名   # 会自动为创建的用户指定主目录、系统shell版本，会在创建时输入用户密码
useradd 用户名   # 需要使用参数选项指定上述基本设置，如果不使用任何参数，则创建的用户无密码、无主目录、没有指定shell版本
userdel 用户名   

```
```
useradd命令：创建用户 
useradd [选项] 登录名 
-u，–uid UID：指定UID，默认是上一个用户的UID+1 
-g，–gid GID：指定基本组ID，此组得事先存在； 
-G，–groups GROUP1[,GROUP2,……[,GROUPSN]]：指明用户所属的附加组，多个组之间用逗号分隔。 
-c，–comment COMMENT：指明注释信息 
-d，–home HOME_DIR：以指定路径为用户的家目录；通过复制/etc/skel此目录并重命名实现；指定的家目录路径如果事先存在，则不会为用户复制环境配置文件。 
-s，–shell SHELL：指定用户的默认shell，可用的所有shell列表存储在/etc/shells文件中； 
-r，–system：创建系统用户 
-M：不为用户创建主目录 
-f，–incative INACTIVE：在密码过期后，账户被彻底禁用之前的天数，0表示立即禁用，-1表示禁用该功能。 
注意：创建用户时的诸多默认设定配置文件为/etc/login.defs

useradd -D：显示创建用户的默认选项配置； 
useradd -D 选项：修改默认选项的值； 
修改的结果保存于/etc/default/useradd文件中；可以直接修改此文件来实现。

usermod命令：修改用户属性 
usermod [选项] 登录名 
-u，–uid UID：修改用户的ID为此处指定的新UID； 
-g，–group GROUP：修改用户所属的基本组；此组得事先存在； 
-G, –groups GROUP1[,GROUP2,…[,GROUPN]]]：修改用户所属的附加组，原来的附加组会被覆盖； 
-a, –append：与-G一同使用(即 -a -G 附加组名)，用于用户追加新的附加组； 
-c，–comment COMMENT：修改注释信息； 
-d，–home HOME_DIR：修改用户的家目录；用户原有的文件不会被转移至新位置； 
-m，–move-home：只能与-d选项一同使用，用于将原来的家目录移动为新的家目录； 
-l，–login NEW_LOGING：修改用户名； 
-s, –shell SHELL：修改用户的默认shell； 
-L，–lock：锁定用户密码；即在用户原来的密码字符串之前添加一个”！”； 
-U，–unlock：解锁用户密码，

userdel命令：删除用户， 
userdel [选项] 登录名 
-r：删除用户时一并删除其家目录和用户邮箱；

id命令：显示用户的真实和有效的UID和GID 
id [OPTION]… [USERNAME] 
-u：仅显示有效的UID； 
-g：仅显示用户的基本组的ID； 
-G：仅显示用户所属的所有组的ID； 
-n：显示名字而非ID；一般与g一起使用：-ng

su命令：switch user **不指定用户就默认切换到root**
登录式切换：会通过重新读取目标用户的配置文件来重新初始化 
su - USERNAME   **可以获取该用户的环境变量**

非登录式切换：不会读取目标用户的配置文件进行初始化 
su USERNAME

注意：管理员可无密码切换至其它任何用户；其它用户在切换用户时必须输入密码。

-c “COMMAND”：仅以指定用户的身份运行此处指定的命令 

groupadd命令：添加用户组
groupadd [选项] 组名
-g GID：指定组ID；

groupmod命令：修改组属性
groupmod [选项] 组名称
-g GID：修改后的组ID；
-n 新组名：修改后的组名；

groupdel命令：删除用户组
groupdel [选项] 组名
此命令仅适用于删除那些 "不是任何用户初始组" 的群组

```



### 单网卡配置多个IP地址

配置文件 `/etc/network/interfaces`
```
 auto ens3                         #自动启用
    iface ens3 inet static         #static或是dhcp方式
    address 192.168.1.88           #IP地址
    netmask 255.255.255.0          #子网掩码
    gateway 192.168.1.1            #默认网关

auto ens3:0                        #自动启用
    iface ens3:0 inet static       #static或是dhcp方式
    address 192.168.1.89           #IP地址
    netmask 255.255.255.0          #子网掩码

auto ens3:1                        #自动启用
    iface ens3:1 inet static       #static或是dhcp方式
    address 192.168.1.90           #IP地址
    netmask 255.255.255.0          #子网掩码


ifup ens3:0 ens3:1
```

### 第三篇: FTP服务器
安装vsftpd,配置文件 /etc/vsftpd.conf，并对默认文件夹 /srv/ftp给予合适权限
```
anonymous_enable=YES    #设置匿名可登录
```
