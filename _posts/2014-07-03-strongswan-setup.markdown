---
layout: post
title:  "在OpenVZ VPS上搭建和使用Strongswan"
date:   2014-07-03 10:35:06
categories: L2TP IPSec
---

由于OpenVZ的限制以前使用Racoon和xl2tpd组合的方式无法在这类VPS使用，只好购买价格更贵的Xen的。开源的世界总是三十年河东，三十年河西， 从Openswan到Racoon, 现在轮到Strongswan了。得益于灵活的插件体系结构，Strongswan具备良好的扩展性， 架设一个L2TP/IPSec的服务器你再也不需要其他的组件，Strongswan就一站式提供了。 尤其是[kernel-libipsec] 插件使得在OpenVZ这样的受限的VPS上也能轻松的使用L2TP/IPec了, 一般的IPSec/IKE实现都要在kernel的IPsec stack中实现，libipsec提供另一种实现方式， 即利用TUN设备，Strongswan的守护程序在收到ESP的UDP包后,通过libipsec进行解密，解密后的明文注入到TUN设备来实现网络连接。

大概了解了基本原理，下面就开始动手安装吧，首先记得去VPS的控制面板下启用TUN/TAP设备， 安装过程稍微有点麻烦，首先需要最新的Strongswan(5.1.3)版本，各发行版不一定有最新的版本， 所以需要手动编译。这里简单说一下在CentOS 6 上的安装, 我习惯用yum 编译安装.

## 编译安装

    # rpm -ivh http://mirror.es.its.nyu.edu/epel/6/i386/epel-release-6-8.noarch.rpm
    # rpm install yum-utils
    $ yumdownloader --source strongswan
    $ rpm -ivh strongswan-5.1.3-1.el6.src.rpm


修改 rpmbuild/SPECS/strongswan.spec , diff 如下：

{%highlight diff%}
--- rpmbuild/SPECS/strongswan.spec.bak	2014-04-15 11:20:24.000000000 +0400
+++ rpmbuild/SPECS/strongswan.spec	2014-06-29 12:21:20.061817133 +0400
@@ -129,7 +129,8 @@
     --enable-eap-radius \
     --enable-curl \
     --enable-eap-identity \
-    --enable-cmd
+    --enable-cmd \
+    --enable-kernel-libipsec
 make %{?_smp_mflags}
 
 %install
@@ -204,6 +205,8 @@
 %{_libdir}/%{name}/libtls.so.0.0.0
 %{_libdir}/%{name}/libpttls.so.0
 %{_libdir}/%{name}/libpttls.so.0.0.0
+%{_libdir}/%{name}/libipsec.so.0
+%{_libdir}/%{name}/libipsec.so.0.0.0
 %{_libdir}/%{name}/lib%{name}.so.0
 %{_libdir}/%{name}/lib%{name}.so.0.0.0
 %dir %{_libdir}/%{name}/plugins
@@ -252,6 +255,7 @@
 %{_libdir}/%{name}/plugins/lib%{name}-dhcp.so
 %{_libdir}/%{name}/plugins/lib%{name}-curl.so
 %{_libdir}/%{name}/plugins/lib%{name}-eap-identity.so
+%{_libdir}/%{name}/plugins/lib%{name}-kernel-libipsec.so
 %dir %{_libexecdir}/%{name}
 %{_libexecdir}/%{name}/_copyright
 %{_libexecdir}/%{name}/_updown
@@ -335,6 +339,7 @@
 %{_sysconfdir}/%{name}/%{name}.d/charon/xauth-generic.conf
 %{_sysconfdir}/%{name}/%{name}.d/charon/xauth-pam.conf
 %{_sysconfdir}/%{name}/%{name}.d/charon/xcbc.conf
+%{_sysconfdir}/%{name}/%{name}.d/charon/kernel-libipsec.conf
 %{_sysconfdir}/%{name}/%{name}.d/imcv.conf
 %{_sysconfdir}/%{name}/%{name}.d/pacman.conf
 %{_sysconfdir}/%{name}/%{name}.d/starter.conf
@@ -397,6 +402,7 @@
 %{_datadir}/%{name}/templates/config/plugins/xauth-generic.conf
 %{_datadir}/%{name}/templates/config/plugins/xauth-pam.conf
 %{_datadir}/%{name}/templates/config/plugins/xcbc.conf
+%{_datadir}/%{name}/templates/config/plugins/kernel-libipsec.conf
 %{_datadir}/%{name}/templates/config/%{name}.conf
 %{_datadir}/%{name}/templates/config/%{name}.d/attest.conf
 %{_datadir}/%{name}/templates/config/%{name}.d/charon-logging.conf

{%endhighlight%}

修改完之后进行打包和安装
     
      $ rpmbuild -bb rpmbuild/SPECS/strongswan.spec
      # rpm -ivh rpmbuild/RPMS/i686/strongswan-5.1.3-1.el6.i686.rpm 


## 配置

这里的设置使用PSK+Xauth的认证方式

/etc/strongswan/ipsec.conf
{% highlight bash %}
config setup
	# strictcrlpolicy=yes
	# uniqueids = no
        # charondebug="ike 1, chd 1, knl 1, cfg 0"

conn %default
	ikelifetime=24h
	keylife=8h
	rekeymargin=3m
	keyingtries=1
        fragmentation=yes
        left=%defaultroute
        leftsubnet=0.0.0.0/0

conn nat-t 
        right=%any
        rightsourceip=10.8.0.0/24
        leftauth=psk
        rightauth=psk
        rightauth2=xauth
        auto=add
{% endhighlight %}

/etc/strongswan/ipsec.secrets
{% highlight bash %}

 : PSK 'mypsk'    #预共享密钥

username : XAUTH "password"  #用户名和密码
{% endhighlight %}

/etc/strongswan/strongswan.d/charon.conf

{% highlight bash %}
    ...
    #这里省略了其他的默认配置， 指定DNS服务器
    dns1=8.8.8.8
    dns2=8.8.4.4
    ...
{% endhighlight %}

启动服务

     # service strongswan start

## 配置转发和SNAT
   
   启用转发

      # vi /etc/sysctl.conf # 修改 net.ipv4.ip_forward = 1
      # sysctl -p
  
   启用SNAT

      # iptables -t nat -A POSTROUTING -j SANT --to-source a.b.c.d # VPS的外部IP地址


## 客户端配置

   Android手机: VPN 配置类型中选择IPSec Xautu PSK, 预共享密钥和用户名密码为/etc/strongswan/ipsec.secrets中的设置
   
   Mac OSX: VPN类型选用CISCO IPSEC， 预共享密钥和用户名密码同上，群组名称记得留空


[kernel-libipsec]: http://wiki.strongswan.org/projects/strongswan/wiki/Kernel-libipsec

