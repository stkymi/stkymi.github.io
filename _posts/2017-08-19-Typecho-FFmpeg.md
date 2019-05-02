---
layout: post
title: "Typecho | FFmpeg" 
date:   2017-08-19 10:35:06
categories: 
---

<!-- more -->

### 第一篇:  Mariadb 的账户管理
安装运行`mariadb`
```
mysql_secure_installation     #初始化设置
mysql -u root -p              #以root账户登入
create/drop database <库名>; 
show databases;
```
数据库默认的管理员用户只能在具有系统管理员权限的程序中使用
```
MariaDB [(none)]> CREATE USER 'by2'@'localhost' IDENTIFIED BY 'my password';
MariaDB [(none)]> GRANT ALL ON *.* TO 'by2'@'localhost';
MariaDB [(none)]> flush privileges;
```
### 第二篇: Typecho 伪静态地址、开启HTTPS
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
编辑站点根目录下的文件`config.inc.php`,添加配置
```
/** 开启HTTPS */
define('__TYPECHO_SECURE__',true);
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
### 第四篇: FFmpeg

转码
```
ffmpeg -i i.flv o.mp4
```

合并
```
ffmpeg -i video.mp4 -i audio.m4a -c copy o.mp4
```

剪辑
```
ffmpeg -ss 00:00:00 -t 00:00:30 -i i.mp4 -vcodec copy -acodec copy o1.mp4
ffmpeg -ss 00:00:30 -t 00:00:30 -i i.mp4 -vcodec copy -acodec copy o2.mp4
```

在list.txt文件中，对要合并的视频片段进行描述
```
file ./o1.mp4
file ./o2.mp4
```

合并
```
ffmpeg -f concat -i list.txt -c copy output.mp4
```

音量延迟
```
ffmpeg -i i.m4a  -filter_complex adelay="1000|1000"  o.m4a  #左右声道均延迟1000毫秒
```
