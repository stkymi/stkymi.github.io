---
layout: post
title:  "企业服务器部署"
date:   2018-03-16 10:35:06
categories:
---

### 第一篇: 校园网 DNS 服务器
添加需要解析的域名
```
zone "example.com" {
    type master; #主服务器（Master）
    file "/etc/bind/db.example.com";
};
```
在注册商填写 Glue Record，表明Name Server处填写的ns.example.com指向此IP，并编写 /etc/bind/db.example.com
```
; BIND data file for example.com;
$ORIGIN example.com.  #以下记录若不以.结尾，则在尾部添加补全此域名
$TTL    7200  #表示默认的缓存时间
@       IN      SOA     ns.example.com. i.example.com. (
                          2017090607       ; Serial   #当前 Zone 文件的版本的序号
                                 7200      ; Refresh  #从属服务器的缓存刷新时间
                                 7200      ; Retry    #从属服务器刷新失败后，等待再次刷新的时间
                               1209600     ; Expire   #当前 Zone 的设定在从属服务器刷新失败一定时间后，自动作废
                                 7200 )    ; Negative Cache TTL  #从属服务器中无效响应的缓存时间
;
@       IN      NS      ns.example.com.
@       IN      A       ***.***.***.***
ns      IN      A       ***.***.***.***
@       IN      TXT     "============="
@       IN      MX      5 mail.example.com.
```
配置从属服务器，并在主服务器对应域名添加 `allow-transfer { ***.***.***.***; };` 以及NS记录，注册商填写 Glue Record
```
zone "example.com" IN {
    type slave;  从属服务器（Slave）
    file "/etc/bind/db.example.com";  保存从 master 过来的数据,不存在则不保存
    masters { ***.***.***.***; };
    allow-notify { 127.0.0.1; ***.***.***.***; ***.***.***.***; };
};
```
### 第二篇: 校园网 DHCP 服务器
配置接口 /etc/default/isc-dhcp-server 以及静态地址 /etc/network/interfaces
```
apt install isc-dhcp-server   


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
配置全局网络与地址池网络 `/etc/dhcp/dhcpd.conf`

### 第三篇: FTP服务器
安装vsftpd,配置文件 /etc/vsftpd.conf，并对默认文件夹 /srv/ftp给予合适权限
```
anonymous_enable=YES    #设置匿名可登录
```