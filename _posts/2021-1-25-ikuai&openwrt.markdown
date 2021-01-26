---
layout: post
title:  "ikuai | openwrt"
date:   2021-01-25 23:23:06
categories:
---

<!--more-->
光猫设置为桥接模式，建议关闭DHCP服务。家里的TP-link光猫默认管理地址为192.168.1.1

### 安装镜像
下载img镜像，在PE环境将磁盘分区删除（不需要再创建），利用工具将img镜像写入磁盘。重启

openwrt默认管理ip为192.168.1.1 若家庭局域网不属于此网段，即修改 /etc/config/network 的ip地址，连接到主路由的lan口，即可在浏览器访问管理界面。
编辑LAN接口，网关和DNS填写主路由的ip，然后openwrt就可以连网了。opkg update，然后在系统->软件界面，搜索chinese，安装base环境的中文包。

### 旁路由
设置主路由的DHCP服务器，网关和DNS都设置为旁路由的ip。或者直接使用旁路由作为DHCP服务器

如果安装ikuai，则添加默认静态路由指向主路由，DNS转发至拔号得到的运营商dns服务器；如果安装openwrt，关键是配置dns转发。

### 单臂路由
ikuai的单臂路由会自动将虚拟接口划分vlan，似乎不怎么好用。

openwrt添加新接口，不开启桥接，物理接口选择与LAN同样的接口，协议为PPPoE，防火墙设置里分配到wan区域（否则拔号成功后不能上网）。
LAN口的物理设置取消桥接。网关和DNS留空。系统防火墙设置里，lan和wan的出入站和转发全部接受。将光猫过来的网线从主路由的WAN口改接到LAN口，openwrt拔号成功后就能上网了。

查看openwrt概览，在ipv4上游可看到拔号获得的网络信息（包含DNS地址）。可见拔号的话就不用设置dns转发，在/etc/hosts文件添加域名与ip对应关系，就不用在客户端修改hosts文件了。
目前使用原来的主路由（现在作为具备有线和无线功能的AP）的DHCP功能，将网关和DNS指向单臂路由。这一点和旁路由设置的一样，只是拔号的设备和旁路由的方式相比不一样了。
若要使用openwrt作为DHCP服务器，修改 /etc/config/dhcp文件。或在LAN接口配置。
