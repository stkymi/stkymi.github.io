---
layout: post
title: "桌面运维与网络工程" 
date:   2019-04-20 10:35:06
categories: 
---

<!-- more -->

## 当安装程序运行到选择安装时按下 `Shift+F10` 启动命令窗口

### DiskPart
命令`Diskpart` 进入环境，其提示符为 `DISKPART>`
常用的命令有：
```
list
select
clean
create
active
format quick
```
注意：DiskPart必须选对目标，当前被选中的磁盘或分区前面会有`*`标记，可以用`list`查看

### 磁盘检查 

进系统时蓝屏，PE查看系统盘打不开、格式为RAW。DiskGenius备份数据后，运行`chkdsk C:/F`修复损坏的文件

### VMware 桥接模式

首先查看本地以太网属性是否安装VMware Bridge Protocol：`控制面板>>网络和Internet>>网络连接`选择实际使用的网卡，右键查看属性是否安装并启用VMware Bridge Protocol，然后查看VMware虚拟网络编辑器的桥接模式设置：`编辑>>虚拟网络编辑器>>桥接模式`桥接目标修改为本地实际的网卡适配器，最后刷新（禁用再启用）虚拟机的本地连接

### 运营商ISP

```
中国电信  telecomadmin  nE7jA%5m
中国移动  CMCCAdmin    aDm8H%MdA
```
