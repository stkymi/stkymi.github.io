---
layout: post
title:  "IKEv2"
date:   2017-09-26 10:35:06
categories:
---

### 生成证书

#### 生成CA根证书

1、生成一个私钥
```
strongswan pki --gen --outform pem > ca.key.pem
```
2、基于这个私钥自己签一个CA根证书
```
strongswan pki --self --in ca.key.pem --dn "C=CN, O=ZhuZhou, CN=StrongSwan CA" --ca --lifetime 3650 --outform pem > ca.cert.pem
```
#### 生成服务器证书
1、生成一个私钥
```
strongswan pki --gen --outform pem > server.key.pem
```
2、用 CA 根证书给自己发一个服务器证书：先从我们刚生成的私钥里把公钥提取出来，然后用公钥去参与服务器证书签发
```
strongswan pki --pub --in server.key.pem --outform pem > server.pub.pem
strongswan pki --issue --lifetime 3650 --cacert ca.cert.pem --cakey ca.key.pem --in server.pub.pem --dn "C=CN, O=ZhuZhou, CN=IP or domain" --san="IP or domain" --flag serverAuth --flag ikeIntermediate --outform pem > server.cert.pem
```
#### 安装证书
```
cp ca.key.pem /etc/strongswan/ipsec.d/private/
cp ca.cert.pem /etc/strongswan/ipsec.d/cacerts/
cp server.cert.pem /etc/strongswan/ipsec.d/certs/
cp server.pub.pem /etc/strongswan/ipsec.d/certs/
cp server.key.pem /etc/strongswan/ipsec.d/private/
```
### 配置Strongswan
ipsec 配置文件`/etc/strongswan/ipsec.conf`
```
config setup
	uniqueids = no  #如果同一个用户在不同的设备上重复登录,yes 断开旧连接,创建新连接;no 保持旧连接,并发送通知; never 保持旧连接, 但不发送通知.

conn %default                        #定义连接项, 命名为 %default 所有连接都会继承它
	keyexchange=ike           
	left=%any                    #服务端公网ip, %any表示从本地ip地址表中取.
	leftsubnet=0.0.0.0/0         #服务器端子网, 如果为客户端分配虚拟 IP 地址，那表示之后要做 iptables 转发，此处就必须是用魔术字
	right=%any                   #客户端公网ip, %any表示从本地ip地址表中取.
 	rightsourceip=10.0.0.0/24    #分配给客户端的虚拟 ip 段
	leftca=ca.cert.pem           #服务器 CA 证书
	leftcert=server.cert.pem     #服务器证书
        ike=aes256-sha256-modp1024,3des-sha1-modp1024,aes256-sha1-modp1024!
	esp=aes256-sha256,3des-sha1,aes256-sha1!
        leftid=IP or @domain          #远程ID，
        rightid=%any                  #本地ID，
        leftdns=8.8.8.8
        rightdns=8.8.8.8
        
conn IKEv2-Pubkey-EAP	
	leftauth=pubkey               #服务器使用证书认证
       	rightauth=eap-mschapv2        #客户端使用 EAP 扩展认证
	leftsendcert=always           #是否发送服务器证书到客户端	
	rightsendcert=never           #客户端是否发送证书到服务器
	auto=add                      #定义 strongswan 启动时该连接的行为。start 是启动; route 是添加路由表，有数据通过就启动; add 是添加连接类型但不启动; ignore 是当它不存在。默认是 ignore。

conn IKEv2-PSK
        leftauth=psk
        rightauth=psk
        auto=add
        
conn IKEv1-PSK-XAUTH
	keyexchange=ikev1
	fragmentation=yes
	leftauth=psk
	rightauth=psk
	rightauth2=xauth
	auto=add
```

密码认证文件 `/etc/strongswan/ipsec.secrets`

```
: RSA server.key.pem   #使用证书验证时的服务器端私钥
: PSK "预设加密密钥"    #使用预共享密钥
用户名 : EAP "密码"     #EAP 方式
用户名 : XAUTH "密码"   #XAUTH 方式, 只适用于 IKEv1
```
内核转发与iptables配置
修改/etc/sysctl.conf 并执行命令 `sysctl -p`
```
net.ipv4.ip_forward = 1
```
```
iptables -t nat -A POSTROUTING -s 10.1.0.0/16 -o eth0 -j MASQUERADE
```

### 配置客户端
安装 CA 根证书 ca.cert.pem，以验证服务器的真实性；Windows将 ca.cert.pem 重命名为 ca.cert.crt，安装至“受信任的根证书颁发机构”。
