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
	uniqueids = no

conn %default
	keyexchange=ike              #ikev1 或 ikev2 都用这个
	left=%any                    #服务器端标识,%any表示任意
	leftsubnet=0.0.0.0/0,::/0    #服务器端虚拟ip, 0.0.0.0/0表示通配.
	right=%any                   #客户端标识,%any表示任意

conn IKE-BASE
	leftca=ca.cert.pem           #服务器端 CA 证书
	leftcert=server.cert.pem     #服务器端证书
	rightsourceip=10.10.0.0/24,fec3::/120   #分配给客户端的虚拟 ip 段

conn IKEv2-EAP
	also=IKE-BASE
	keyexchange=ikev2
	ike=aes256-sha256-modp1024,3des-sha1-modp1024,aes256-sha1-modp1024!
	esp=aes256-sha256,3des-sha1,aes256-sha1!
	rekey=no                     #服务器对 Windows 发出 rekey 请求会断开连接
	leftid=VPS的外网ip地址          #远程ID，也就是服务器IP
	leftauth=pubkey
	leftsendcert=always
	#leftfirewall=yes
	right=%any
	rightfirewall=yes
	#rightsourceip=192.168.2.1/24
	rightsendcert=never
	rightauth=eap-mschapv2
	eap_identity=%any
	dpdaction=clear
	fragmentation=yes
	auto=add

conn IPSec-IKEv1-PSK
	also=IKE-BASE
	keyexchange=ikev1
	fragmentation=yes
	leftauth=psk
	rightauth=psk
	rightauth2=xauth
	auto=add
```

密码认证文件 `/etc/strongswan/ipsec.secrets`

```
#使用证书验证时的服务器端私钥
#格式 : RSA <private key file> [ <passphrase> | %prompt ]
: RSA server.key.pem

#使用预设加密密钥, 越长越好
#格式 [ <id selectors> ] : PSK <secret>
%any : PSK "预设加密密钥"

#EAP 方式, 格式同 psk 相同
用户名 : EAP "密码"

#XAUTH 方式, 只适用于 IKEv1
#格式 [ <servername> ] <username> : XAUTH "<password>"
用户名 : XAUTH "密码"
```
