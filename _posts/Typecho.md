---
layout: post
title: "Typecho" 
date:   2017-08-19 10:35:06
categories: 
---

<!-- more -->

### 第一篇:  Mariadb 的账户管理
数据库默认的管理员用户只能在具有系统管理员权限的程序中使用
```
MariaDB [(none)]> CREATE USER 'by2'@'localhost' IDENTIFIED BY 'my password';
MariaDB [(none)]> GRANT ALL ON *.* TO 'by2'@'localhost';
MariaDB [(none)]> flush privileges;
```
### 第二篇: Typecho 伪静态地址
需要 Rewrite 权限。根目录创建 .htaccess
```
 <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /  #根目录
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.*)$ /index.php/$1 [L]  #涉及目录
    </IfModule>
```
### 第三篇: 开启bbr
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```
保存配置文件sysctl -p,检查确认
```
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
lsmod | grep bbr
```
