---
layout: post
title:  "IKEv2"
date:   2017-09-26 10:35:06
categories:
---
OpenVZ需要开启TUN，并安装libipsec插件；CentOS使用`strongswan`命令、有strongswan文件夹，Debian使用`ipsec`命令、没有strongswan文件夹并且需要安装pki和xauth插件
### 证书（Windows必选，Iphone可选）

#### 生成CA根证书：生成一个私钥，基于这个私钥自己签一个CA根证书
```
strongswan pki --gen --outform pem > ca.key.pem
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
	leftca=ca.cert.pem          
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
```

### 配置客户端
安装 CA 根证书 ca.cert.pem，以验证服务器的真实性；Windows将 ca.cert.pem 重命名为 ca.cert.crt，安装至“受信任的根证书颁发机构”,适配器属性选择“需要加密”和“在远程网络上使用默认网关”。
