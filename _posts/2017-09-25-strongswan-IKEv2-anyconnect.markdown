---
layout: post
title:  "IKEv2 | AnyConnect"
date:   2017-09-26 10:35:06

---

加密相关的预备知识：对称加密和非对称加密。

对称加密 ： 加密和解密数据使用同一个密钥。这种加密方式的特点是速度很快，常见对称加密的算法有 AES；
非对称加密： 加密和解密使用不同的密钥，这两个密钥形成有且仅有唯一的配对，叫公钥和私钥。数据用公钥加密后必须用私钥解密，数据用私钥加密后必须用公钥解密。一般来说私钥自己保留好，把公钥公开给别人（一般公钥不会单独出现，而是会写进证书中），让别人拿自己的公钥加密数据后发给自己，这样只有自己才能解密。 这种加密方式的特点是速度慢，CPU 开销大，常见非对称加密算法有 RSA。

CA证书的相关知识：
CA证书是由CA（Certification Authority）机构发布的数字证书。其内容包含：电子签证机关的信息、公钥用户信息、公钥、签名和有效期。这里的公钥服务端的公钥，这里的签名是指：用hash散列函数计算公开的明文信息的信息摘要，然后采用CA的私钥对信息摘要进行加密，加密完的密文就是签名。
即：证书 =  公钥 + 签名 +申请者和颁发者的信息。
客户端中因为在操作系统中就预置了CA的公钥，所以支持解密签名（因为签名使用CA的私钥加密的）



服务端拥有一对非对称密钥：B_公钥和B_私钥。Https单向认证详细过程如下：
（1）客户端发起HTTPS请求，将SSL协议版本的信息发送给服务端。
（2）服务端去CA机构申请来一份CA证书，在前面提过，证书里面有服务端公钥和签名。将CA证书发送给客户端
（3）客户端读取CA证书的明文信息，采用相同的hash散列函数计算得到信息摘要（hash目的：验证防止内容被修改），然后用操作系统带的CA的公钥去解密签名（因为签名是用CA的私钥加密的），对比证书中的信息摘要。如果一致，则证明证书是可信的，然后取出了服务端公钥
（4）客户端生成一个随机数（密钥F），用刚才等到的服务端B_公钥去加密这个随机数形成密文，发送给服务端。
（5）服务端用自己的B_私钥去解密这个密文，得到了密钥F
（6）服务端和客户端在后续通讯过程中就使用这个密钥F进行通信了。和之前的非对称加密不同，这里开始就是一种对称加密的方式

HTTPS在保证数据安全传输上使用对称加密和非对称加密相结合的方式来进行的，简单来说就是通过一次非对称加密算法进行了最终通信密钥的生成、确认和交换，然后在后续的通信过程中使用最终通信密钥进行对称加密通信。之所以不是全程非对称加密，是因为非对称加密的计算量大，影响通信效率。

```
作者：JasonZ
链接：https://juejin.cn/post/6844903809068564493
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### Strongswan
OpenVZ需要开启TUN，并安装libipsec插件,然而Debian的apt并不提供此插件，因此OpenVZ架构下的Debian只能编译安装（编译后均使用`ipsec`和`swanctl`命令）；若是包安装方式：CentOS使用`strongswan`命令、有strongswan文件夹，Debian使用`ipsec`命令、没有strongswan文件夹并且需要安装pki和eap-mschapv2插件

## 务必使用 centos-6-x86_64-minimal.tar.gz 
低版本系统编译新版strongswan非常简单，但是编译新版ocserv非常困难。所以通过epel-release实现。CentOS6停留在了0.12.6版本

#### 编译安装

安装依赖
```
yum install pam-devel openssl-devel gcc gcc-c++  gmp 
sudo apt-get install libpam0g-dev libssl-dev make gcc
```
```
wget --no-check-certificate https://download.strongswan.org/strongswan.tar.gz
tar -xzf strongswan.tar.gz
cd strongswan-*
```
编译：OpenVZ必须添加 `--enable-kernel-libipsec`

```
./configure --enable-kernel-libipsec --enable-openssl --disable-gmp --enable-eap-identity --enable-eap-mschapv2 
```
~~仅包含最简单的PSK预共享密钥认证，~~ 这里使用openssl替换gmp,因为Debian不带gmp包，也要编译

```
make
sudo make install 
```

默认安装到`/usr/local`目录，配置文件在 `/usr/local/etc`

```
sudo ipsec start/stop/status
```

### 证书（Windows必选，Iphone、Android可选）

#### ~~生成CA根证书：生成一个私钥，基于这个私钥自己签一个CA根证书~~
```
ipsec pki --gen --outform pem > ca.key.pem
ipsec pki --self --in ca.key.pem --dn "C=CN, O=ZhuZhou, CN=StrongSwan CA" --ca --lifetime 3650 --outform pem > ca.cert.pem
```
#### ~~生成服务器证书~~
~~1、生成一个私钥~~
```
ipsec pki --gen --outform pem > server.key.pem
```
~~2、用 CA 根证书给自己发一个服务器证书：先从我们刚生成的私钥里把公钥提取出来，然后用公钥去参与服务器证书签发~~
```
ipsec pki --pub --in server.key.pem --outform pem > server.pub.pem
ipsec pki --issue --lifetime 3650 --cacert ca.cert.pem --cakey ca.key.pem --in server.pub.pem --dn "C=CN, O=ZhuZhou, CN=IP or domain" --san="IP or domain" --outform pem > server.cert.pem
```
#### 安装证书
编译的路径为`/usr/local/etc/ipsec.d`
Windows使用IKEv2的最大难题之一就是证书了吧，总结了经验教训下来，最重要的一点就是，遇到问题要查看log，不要瞎琢磨。
泛域名证书是支持的，在客户端打开服务器部署的证书，查看证书路径，一般为根证书->中间证书->服务器证书。那么，就要把根证书和中间证书（可以从Windows系统中导出）放到ipsec.d的cacerts文件夹中，否则会提示IKE身份验证凭证不可接受。
### 更新：通过Caddy从Let's Encrypt获取服务器证书
链接服务器证书和私钥
```
ln -s /root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain.com/domain.com.crt /etc/strongswan/ipsec.d/certs/server.crt
ln -s /root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain.com/domain.com.key /etc/strongswan/ipsec.d/private/private.key
```
Windows：必须提供Let's Encrypt中间证书 Let’s Encrypt Authority X3以及根证书DST Root CA X3，否则会出现`错误13801:IKE身份验证凭证不可接受。` 中间证书和根证书都可以在官网https://letsencrypt.org/zh-cn/certificates/ 获取。
```
cd /etc/strongswan/ipsec.d/cacerts
wget https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem 
```
### 配置Strongswan
自5.7版本开始使用`swanctl.conf`配置文件（编译添加 --enable-swanctl ），但目前仍兼容旧的配置方法

ipsec 配置文件`/etc/strongswan/ipsec.conf`
```
config setup
uniqueids = never

conn %default                       
	keyexchange=ike           
	left=%any                    
	leftsubnet=0.0.0.0/0        
	right=%any                  
 	rightsourceip=172.16.0.0/16
	leftcert=server.crt
	leftid=domain
	rightid=%any                 
        leftdns=8.8.8.8
        rightdns=8.8.8.8          
conn IKEv2-PSK-PSK
        leftauth=psk
        rightauth=psk
        auto=add
conn IKEV2-Windows-eap-mschapv2
	dpdaction=clear
	dpddelay=60s
	rekey=no
	fragmentation=yes
	ike=aes256-sha1-modp1024!
	leftauth=pubkey
	leftsendcert=always
	rightauth=eap-mschapv2
	rightsendcert=never
	eap_identity=%identity
	auto=add

#关于leftid，如果只编译psk认证，也没有部署服务器证书，这里写什么都是可以的，只要不包含特殊字符
#添加了eap-mschapv2和证书后，必须写成证书的域名，否则Iphone的PSK方式都无法连接了，Android不受影响
#leftid完全不影响Windows的eap-mschapv2连接。
#如果不定义ike=aes256-sha1-modp1024! Windows连接提示策略匹配错误，试了几个，Windows只有这个算法可以连接
#如果不定义eap_identity=%identity Windows用保存的密码连接总是提示不正确，要重新输一遍密码。但是定义之后，其实任何用户名都是可以的，只要密码正确即可
#编译时就要添加 --enable-eap-identity

```

密码认证文件 `/etc/strongswan/ipsec.secrets`

```
: RSA private.key  
: PSK "The key"    
username : EAP "password"  

#PSK密钥只能有一把，EAP可以添加多行
```
内核转发与iptables配置
修改/etc/sysctl.conf 并执行命令 `sysctl -p`
```
net.ipv4.ip_forward = 1
```
```
iptables -t nat -A POSTROUTING -s 172.16.0.0/12 -o venet0 -j MASQUERADE
```
```
service iptables save  # IPv4规则会保存到 /etc/sysconfig/iptables 文件,保存后系统重启会自动加载
chkconfig iptables on  # 开机启动
```
Windows要在网卡属性、网络属性、高级中勾选“在远程网络上使用默认网关” Windows 10 还可以设置VPN代理属性，私网地址等不走VPN路由

### 启动脚本

保存到`/etc/init.d/strongswan`并赋予执行权限
```
#!/bin/sh
#
# strongswan   An implementation of key management system for IPsec
#
# chkconfig:   - 48 52
# description: Starts or stops the Strongswan daemon.

### BEGIN INIT INFO
# Provides: ipsec
# Required-Start: $network $remote_fs $syslog $named
# Required-Stop: $syslog $remote_fs
# Default-Start:
# Default-Stop: 0 1 6
# Short-Description: Start Strongswan daemons at boot time
### END INIT INFO

# Source function library.
. /etc/rc.d/init.d/functions

#exec="/usr/sbin/strongswan"
exec=/usr/local/sbin/ipsec
prog="strongswan"
status_prog="starter"
config="/usr/local/etc/strongswan.conf"

lockfile=/var/lock/subsys/$prog

start() {
    [ -x $exec ] || exit 5
    [ -f $config ] || exit 6
    echo -n $"Starting $prog: "
    daemon $exec start
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    $exec stop
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    stop
    start
}

reload() {
    restart
}

force_reload() {
    restart
}

_status() {
    # run checks to determine if the service is running or use generic status
    status $status_prog
}

_status_q() {
    _status >/dev/null 2>&1
}


case "$1" in
    start)
        _status_q && exit 0
        $1
        ;;
    stop)
        _status_q || exit 0
        $1
        ;;
    restart)
        $1
        ;;
    reload)
        _status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        _status
        ;;
    condrestart|try-restart)
        _status_q || exit 0
        restart
        ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload}"
        exit 2
esac
exit $?
```
```
chkconfig --add strongswan
chkconfig strongswan on
```
### 配置客户端
~~安装 CA 根证书 ca.cert.pem，以验证服务器的真实性；Windows将 ca.cert.pem 重命名为 ca.cert.crt，安装至“受信任的根证书颁发机构”,~~适配器属性选择“需要加密”和“在远程网络上使用默认网关”。

### ocserv
```
auth = "plain[passwd=/etc/ocserv/ocpasswd]"
# 选择登陆方式，这里使用密码登陆并且从 /etc/ocserv/ocpasswd 文件中读取用户名和密码
# 使用证书登录则启用 auth="certificate"
 
tcp-port = 443
udp-port = 443
# 这个代表 TCP和UDP监听的端口 默认443，可以更换其他端口，端口号可分开
 
try-mtu-discovery = true
# 开启以后可以增强VPN性能

# cert-user-oid = 2.5.4.3
# 让服务器读取用户证书,适用于证书登录

server-cert = /etc/pki/ocserv/public/server.crt
server-key = /etc/pki/ocserv/private/private.key
# ca-cert = /etc/pki/ocserv/cacerts/ca.crt
# 服务器证书、私钥和CA证书的位置，这里的CA指的是签发登录证书的CA;用户名和密码登录方式不需要CA证书
# 注意这里的路径最后的位置不能有空格
max-clients = 16
# 允许同时连接的总客户端数量
max-same-clients = 2
# 同账号连接VPN最大客户端数量，0是不作限制
cookie-timeout = 172800
# 连接中断的时间在cookie-timeout数值内可以自动重连

ipv4-network = 172.17.0.0
ipv4-netmask = 255.255.0.0
 
dns = 8.8.8.8
dns = 1.1.1.1
# 服务端的DNS

cisco-client-compat = true
# 使ocserv兼容AnyConnect
```
添加路由表，以下IP段不经过VPN。AnyConnect限制200条路由表
假设客户端使用的是A类的私网地址，那么即使 no-route 添加了10.0.0.0/8 ,这一条路由也不会在Anyconnect客户端的路由表中体现
```
#  route = #全部注释掉route选项，启用 no-route

```
服务器证书
```
ln -s /root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain.com/domain.com.crt /etc/pki/ocserv//public/server.crt
ln -s /root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain.com/domain.com.key /etc/pki/ocserv/private/private.key
```
用户管理
```
ocpasswd -c /etc/ocserv/ocpasswd name      #创建用户名为name的用户，会提示创建密码
ocpasswd -c /etc/ocserv/ocpasswd -d name   #删除用户名为name的用户，无任何提示
ocpasswd -c /etc/ocserv/ocpasswd -l name   #锁定用户名为name的用户，无任何提示
ocpasswd -c /etc/ocserv/ocpasswd -u name   #解锁用户名为name的用户，无任何提示
```
用户证书：用户证书只需 ocserv 信任 CA 即可，因此使用自签发证书。

生成一个私钥，再用这个私钥参与用户证书的签发
```
certtool --generate-privkey --outfile user.key
certtool --generate-certificate --load-privkey user.key --load-ca-certificate ca.crt --load-ca-privkey ca.key --template user.tmpl --outfile user.crt
```
将证书和私钥合成PKCS12格式,会提示创建证书名字和密码。安装证书时需要提供密码
```
certtool --to-p12 --load-privkey user.key --pkcs-cipher 3des-pkcs12 --load-certificate user.crt --outfile user.p12 --outder
```
再修改登录方式为证书验证。无论哪种方式都要启用系统的IP转发功能，否则无法访问网络。OpenVZ需要开启TUN，否则Anyconnect无法登录。
