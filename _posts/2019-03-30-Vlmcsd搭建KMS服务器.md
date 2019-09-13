---
layout: post
title: "Vlmcsd搭建KMS服务器" 
date:   2019-03-30 10:35:06
categories:
---

<!-- more -->

### 搭建KMS服务器

确定linux服务器的CPU架构：`cat /proc/cpuinfo`

`chmod -R`赋予`vlmcsd-x64-musl-static`可执行权限并启动服务`./`

侦听端口是1688，可查看`ss -lnp|grep 1688`

Vlmcs用于检测服务器 `vlmcs ***.com -V`即是显示测试激活的详细信息，`-help`显示所有支持的产品。


### 激活命令

产品列表以及Key
```
https://docs.microsoft.com/zh-cn/windows-server/get-started/kmsclientkeys
https://docs.microsoft.com/zh-cn/DeployOffice/vlactivation/gvlks
https://docs.microsoft.com/zh-cn/previous-versions/office/dn385360(v=office.15)
https://docs.microsoft.com/zh-cn/previous-versions/office/office-2010/ee624355(v=office.14)
```
激活命令：使用管理员权限运行命令提示符CMD
```
slmgr /ipk xxxxx-xxxxx-xxxxx-xxxxx
slmgr /skms kms.03k.org
slmgr /ato
slmgr /xpr
slmgr /dlv
```
激活命令：找到安装目录`C:\Program Files\Microsoft Office\Office16`，确认目录下的`OSPP.VBS`，CD到这个目录

`cscript ospp.vbs /sethst:kms.03k.org`

也可以把ospp.vbs拖进CMD窗口，然后
```
cscript "C:\Program Files\Microsoft Office\Office16\OSPP.VBS" /sethst:kms.03k.org
```

`cscript ospp.vbs /act`


### 自定义系统OEM信息

打开注册表`RegEdit`，定位
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\OEMInformation
```

通过新建字符串值，并修改数值数据来实现自定义OEM信息：
```
Manufacturer -----------制造商
Model ----------------- 设备型号
Logo ------------------ 企业logo图片路径（32位色彩深度、120X120像素、bmp格式）
SupportPhone ---------- 企业服务支持电话
SupportHours ---------- 企业服务支持时间
SupportURL ------------ 企业服务支持网站
```
logo默认路径
```
"C:\Windows\System32\OEMlogo.bmp"
```


