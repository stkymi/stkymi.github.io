---
layout: post
title:  "IKEv2 | AnyConnect"
date:   2017-09-26 10:35:06
categories:
---
### Strongswan
OpenVZ需要开启TUN，并安装libipsec插件；CentOS使用`strongswan`命令、有strongswan文件夹，Debian使用`ipsec`命令、没有strongswan文件夹并且需要安装pki和eap-mschapv2、xauth插件
### 证书（Windows必选，Iphone可选）

#### 生成CA根证书：生成一个私钥，基于这个私钥自己签一个CA根证书
```
ipsec pki --gen --outform pem > ca.key.pem
ipsec pki --self --in ca.key.pem --dn "C=CN, O=ZhuZhou, CN=StrongSwan CA" --ca --lifetime 3650 --outform pem > ca.cert.pem
```
#### 生成服务器证书
1、生成一个私钥
```
ipsec pki --gen --outform pem > server.key.pem
```
2、用 CA 根证书给自己发一个服务器证书：先从我们刚生成的私钥里把公钥提取出来，然后用公钥去参与服务器证书签发
```
ipsec pki --pub --in server.key.pem --outform pem > server.pub.pem
ipsec pki --issue --lifetime 3650 --cacert ca.cert.pem --cakey ca.key.pem --in server.pub.pem --dn "C=CN, O=ZhuZhou, CN=IP or domain" --san="IP or domain" --flag serverAuth --flag ikeIntermediate --outform pem > server.cert.pem
```
#### 安装证书
```
cp ca.key.pem /etc/ipsec.d/private/
cp ca.cert.pem /etc/ipsec.d/cacerts/
cp server.cert.pem /etc/ipsec.d/certs/
cp server.pub.pem /etc/ipsec.d/certs/
cp server.key.pem /etc/ipsec.d/private/

cp ca.key.pem /etc/strongswan/ipsec.d/private/
cp ca.cert.pem /etc/strongswan/ipsec.d/cacerts/
cp server.cert.pem /etc/strongswan/ipsec.d/certs/
cp server.pub.pem /etc/strongswan/ipsec.d/certs/
cp server.key.pem /etc/strongswan/ipsec.d/private/
```

### 更新：通过Caddy从Let's Encrypt获取服务器证书
链接服务器证书和私钥
```
ln -s /root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain.com/domain.com.crt /etc/strongswan/ipsec.d/certs/cert.crt
ln -s /root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain.com/domain.com.key /etc/strongswan/ipsec.d/private/private.key
```
Windows必须提供Let's Encrypt中间证书，否则会出现`错误13801:IKE身份验证凭证不可接受。` 中间证书和根证书都可以在官网https://letsencrypt.org/certificates/ 获取。
```
wget https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem -O /etc/strongswan/ipsec.d/cacerts/intermediate.pem
```
### 配置Strongswan
ipsec 配置文件`/etc/strongswan/ipsec.conf`
```
config setup

conn %default                       
	keyexchange=ike           
	left=%any                    
	leftsubnet=0.0.0.0/0        
	right=%any                  
 	rightsourceip=192.168.0.0/16    
	leftca="C=CN, O=ZhuZhou, CN=StrongSwan CA"      
	leftcert=server.cert.pem    
        leftid=IP or @domain         
        rightid=%any                 
        leftdns=8.8.8.8
        rightdns=8.8.8.8
conn IKEv2-Pubkey-EAP	
	leftauth=pubkey            
       	rightauth=eap-mschapv2       
	leftsendcert=always        
	rightsendcert=never          
	auto=add                     
conn IKEv2-PSK-PSK
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
: RSA server.key.pem  
: PSK "The key"    
username : XAUTH "password"    
username : EAP "password"  
```
内核转发与iptables配置
修改/etc/sysctl.conf 并执行命令 `sysctl -p`
```
net.ipv4.ip_forward = 1
```
```
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o ens3 -j MASQUERADE
iptables-save > /etc/iptables
```
编辑`/etc/network/if-pre-up.d/iptables`，添加启动脚本
```
#!/bin/sh
/sbin/iptables-restore < /etc/iptables
```
必须赋予脚本执行权限，修改配置后应保存

### 配置客户端
安装 CA 根证书 ca.cert.pem，以验证服务器的真实性；Windows将 ca.cert.pem 重命名为 ca.cert.crt，安装至“受信任的根证书颁发机构”,适配器属性选择“需要加密”和“在远程网络上使用默认网关”。

### ocserve
```
auth = "plain[passwd=/etc/ocserv/ocpasswd]"
# 选择登陆方式，plain[passwd=/etc/ocserv/ocpasswd] 代表使用密码登陆并且从 /etc/ocserv/ocpasswd 文件中读取用户名和密码
# 使用证书登录则启用 auth="certificate"
 
tcp-port = 443
udp-port = 443
# 这个代表 TCP和UDP监听的端口 默认443，可以更换其他端口，端口号可分开
 
try-mtu-discovery = true
# 开启以后可以增强VPN性能

cert-user-oid = 2.5.4.3
# 让服务器读取用户证书,适用于证书登录

server-cert = /etc/pki/ocserv/public/server.crt
server-key = /etc/pki/ocserv/private/server.key
ca-cert = /etc/pki/ocserv/cacert/ca.crt
# 服务器证书、私钥和CA证书的位置，这里的CA指的是签发登录证书的CA 
 
max-clients = 16
# 允许同时连接的总客户端数量
max-same-clients = 2
# 同账号连接VPN最大客户端数量，0是不作限制

ipv4-network = 192.168.8.0
ipv4-netmask = 255.255.255.0
 
dns = 8.8.8.8
dns = 1.1.1.1
# 服务端的DNS

cisco-client-compat = true
# 使ocserv兼容AnyConnect
```
用户管理
```
ocpasswd -c /etc/ocserv/ocpasswd name      #创建用户名为name的用户，会提示创建密码
ocpasswd -c /etc/ocserv/ocpasswd -d name   #删除用户名为name的用户，无任何提示
ocpasswd -c /etc/ocserv/ocpasswd -l name   #锁定用户名为name的用户，无任何提示
ocpasswd -c /etc/ocserv/ocpasswd -u name   #解锁用户名为name的用户，无任何提示
```

