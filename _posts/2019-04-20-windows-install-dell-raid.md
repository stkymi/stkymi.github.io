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
