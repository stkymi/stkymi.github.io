---
layout: post
title:  "IKEv2 | AnyConnect"
date:   2017-09-26 10:35:06

---
### Strongswan
OpenVZ需要开启TUN，并安装libipsec插件,然而Debian的apt并不提供此插件，因此OpenVZ架构下的Debian只能编译安装（编译后均使用`ipsec`和`swanctl`命令）；若是包安装方式：CentOS使用`strongswan`命令、有strongswan文件夹，Debian使用`ipsec`命令、没有strongswan文件夹并且需要安装pki和eap-mschapv2插件

#### 编译安装

安装依赖
```
yum install pam-devel openssl-devel gcc gcc-c++ m4 gmp 
apt install libpam0g-dev libssl-dev
```
```
wget http://download.strongswan.org/strongswan.tar.gz
tar -xzf strongswan.tar.gz
cd strongswan-*
```
编译：OpenVZ必须添加 `--enable-kernel-libipsec`

```
./configure --enable-openssl --disable-gmp --enable-kernel-libipsec
```
仅包含最简单的PSK预共享密钥认证，这里使用openssl替换gmp,因为Debian不带gmp包，也要编译，呵呵
```
./configure  --enable-eap-identity --enable-eap-md5 --enable-eap-mschapv2 --enable-eap-tls --enable-eap-ttls --enable-eap-peap --enable-eap-tnc --enable-eap-dynamic --enable-eap-radius --enable-xauth-eap --enable-xauth-pam --enable-dhcp --enable-addrblock --enable-unity --enable-certexpire --enable-radattr --enable-tools --enable-openssl --disable-gmp --enable-kernel-libipsec
```
```
make
make install 或者 make upgrade
```

默认安装到`/usr/local`目录，配置文件在 `/usr/local/etc`

```
ipsec start/stop/status
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
#### ~~安装证书~~
编译的路径为`/usr/local/etc/ipsec.d`
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
Windows：必须提供Let's Encrypt中间证书 Let’s Encrypt Authority X3以及根证书DST Root CA X3，否则会出现`错误13801:IKE身份验证凭证不可接受。` 中间证书和根证书都可以在官网https://letsencrypt.org/zh-cn/certificates/ 获取。
```
cd /etc/strongswan/ipsec.d/cacerts
wget https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem 
```
### 配置Strongswan
自5.7版本开始使用`swanctl.conf`配置文件，但目前仍兼容旧的配置方法

ipsec 配置文件`/etc/strongswan/ipsec.conf`
```
config setup
uniqueids = no

conn %default                       
	keyexchange=ike           
	left=%any                    
	leftsubnet=0.0.0.0/0        
	right=%any                  
 	rightsourceip=192.168.0.0/16
	leftcert=server.cert.pem
	leftid=domain
	rightid=%any                 
        leftdns=8.8.8.8
        rightdns=8.8.8.8          
conn IKEv2-PSK-PSK
        leftauth=psk
        rightauth=psk
        auto=add
conn IKEv2-Pubkey-EAP	
	leftauth=pubkey            
       	rightauth=eap-mschapv2       
	leftsendcert=always        
	rightsendcert=never          
	auto=add           
conn Windows
    keyexchange=ikev2
    ike=aes256-sha256-modp1024,aes256-sha1-modp1024,3des-sha1-modp1024!
    esp=aes256-sha256,aes256-sha1,3des-sha1!
    leftsendcert=always
    leftauth=pubkey
    rightauth=eap-mschapv2
    rightsendcert=never
    eap_identity=%any
    rekey=no
    auto=add
```

密码认证文件 `/etc/strongswan/ipsec.secrets`

```
: RSA server.key.pem  
: PSK "The key"    
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
Debian
```
iptables-save > /etc/iptables
```
编辑`/etc/network/if-pre-up.d/iptables`，添加启动脚本
```
#!/bin/sh
/sbin/iptables-restore < /etc/iptables
```
必须赋予脚本执行权限，修改配置后应保存
```
chmod -R 755 /etc/network/if-pre-up.d/iptables 
```
centOS
```
service iptables save  # IPv4规则会保存到 /etc/sysconfig/iptables 文件,保存后系统重启会自动加载
chkconfig iptables on  # 开机启动
```
### 启动脚本

保存到`/etc/init.d/`目录并赋予执行权限
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
安装 CA 根证书 ca.cert.pem，以验证服务器的真实性；Windows将 ca.cert.pem 重命名为 ca.cert.crt，安装至“受信任的根证书颁发机构”,适配器属性选择“需要加密”和“在远程网络上使用默认网关”。

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

cert-user-oid = 2.5.4.3
# 让服务器读取用户证书,适用于证书登录

server-cert = /etc/pki/ocserv/public/domain.com.crt
server-key = /etc/pki/ocserv/private/domain.com.key
ca-cert = /etc/pki/ocserv/cacerts/ca.crt
# 服务器证书、私钥和CA证书的位置，这里的CA指的是签发登录证书的CA 
# 注意这里的路径最后的位置不能有空格
max-clients = 16
# 允许同时连接的总客户端数量
max-same-clients = 2
# 同账号连接VPN最大客户端数量，0是不作限制
cookie-timeout = 172800
# 连接中断的时间在cookie-timeout数值内可以自动重连

ipv4-network = 192.168.8.0
ipv4-netmask = 255.255.255.0
 
dns = 8.8.8.8
dns = 1.1.1.1
# 服务端的DNS

cisco-client-compat = true
# 使ocserv兼容AnyConnect
```
添加路由表，以下IP段不经过VPN。AnyConnect限制200条路由表
```
#  route = #全部注释掉route选项，启用 no-route
no-route = 1.0.0.0/255.128.0.0
no-route = 1.160.0.0/255.224.0.0
no-route = 1.192.0.0/255.224.0.0
no-route = 10.0.0.0/255.0.0.0
no-route = 14.0.0.0/255.224.0.0
no-route = 14.96.0.0/255.224.0.0
no-route = 14.128.0.0/255.224.0.0
no-route = 14.192.0.0/255.224.0.0
no-route = 27.0.0.0/255.192.0.0
no-route = 27.96.0.0/255.224.0.0
no-route = 27.128.0.0/255.128.0.0
no-route = 36.0.0.0/255.192.0.0
no-route = 36.96.0.0/255.224.0.0
no-route = 36.128.0.0/255.128.0.0
no-route = 39.0.0.0/255.224.0.0
no-route = 39.64.0.0/255.192.0.0
no-route = 39.128.0.0/255.192.0.0
no-route = 42.0.0.0/255.0.0.0
no-route = 43.224.0.0/255.224.0.0
no-route = 45.64.0.0/255.192.0.0
no-route = 47.64.0.0/255.192.0.0
no-route = 49.0.0.0/255.128.0.0
no-route = 49.128.0.0/255.224.0.0
no-route = 49.192.0.0/255.192.0.0
no-route = 54.192.0.0/255.224.0.0
no-route = 58.0.0.0/255.128.0.0
no-route = 58.128.0.0/255.224.0.0
no-route = 58.192.0.0/255.192.0.0
no-route = 59.32.0.0/255.224.0.0
no-route = 59.64.0.0/255.192.0.0
no-route = 59.128.0.0/255.128.0.0
no-route = 60.0.0.0/255.192.0.0
no-route = 60.160.0.0/255.224.0.0
no-route = 60.192.0.0/255.192.0.0
no-route = 61.0.0.0/255.192.0.0
no-route = 61.64.0.0/255.224.0.0
no-route = 61.128.0.0/255.192.0.0
no-route = 61.224.0.0/255.224.0.0
no-route = 100.64.0.0/255.192.0.0
no-route = 101.0.0.0/255.128.0.0
no-route = 101.128.0.0/255.224.0.0
no-route = 101.192.0.0/255.192.0.0
no-route = 103.0.0.0/255.192.0.0
no-route = 103.224.0.0/255.224.0.0
no-route = 106.0.0.0/255.128.0.0
no-route = 106.224.0.0/255.224.0.0
no-route = 110.0.0.0/254.0.0.0
no-route = 112.0.0.0/255.128.0.0
no-route = 112.128.0.0/255.224.0.0
no-route = 112.192.0.0/255.192.0.0
no-route = 113.0.0.0/255.128.0.0
no-route = 113.128.0.0/255.224.0.0
no-route = 113.192.0.0/255.192.0.0
no-route = 114.0.0.0/255.128.0.0
no-route = 114.128.0.0/255.224.0.0
no-route = 114.192.0.0/255.192.0.0
no-route = 115.0.0.0/255.0.0.0
no-route = 116.0.0.0/255.0.0.0
no-route = 117.0.0.0/255.128.0.0
no-route = 117.128.0.0/255.192.0.0
no-route = 118.0.0.0/255.224.0.0
no-route = 118.64.0.0/255.192.0.0
no-route = 118.128.0.0/255.128.0.0
no-route = 119.0.0.0/255.128.0.0
no-route = 119.128.0.0/255.192.0.0
no-route = 119.224.0.0/255.224.0.0
no-route = 120.0.0.0/255.192.0.0
no-route = 120.64.0.0/255.224.0.0
no-route = 120.128.0.0/255.224.0.0
no-route = 120.192.0.0/255.192.0.0
no-route = 121.0.0.0/255.128.0.0
no-route = 121.192.0.0/255.192.0.0
no-route = 122.0.0.0/254.0.0.0
no-route = 124.0.0.0/255.0.0.0
no-route = 125.0.0.0/255.128.0.0
no-route = 125.160.0.0/255.224.0.0
no-route = 125.192.0.0/255.192.0.0
no-route = 127.0.0.0/255.0.0.0
no-route = 139.0.0.0/255.224.0.0
no-route = 139.128.0.0/255.128.0.0
no-route = 140.64.0.0/255.224.0.0
no-route = 140.128.0.0/255.224.0.0
no-route = 140.192.0.0/255.192.0.0
no-route = 144.0.0.0/255.192.0.0
no-route = 144.96.0.0/255.224.0.0
no-route = 144.224.0.0/255.224.0.0
no-route = 150.0.0.0/255.224.0.0
no-route = 150.96.0.0/255.224.0.0
no-route = 150.128.0.0/255.224.0.0
no-route = 150.192.0.0/255.192.0.0
no-route = 152.96.0.0/255.224.0.0
no-route = 153.0.0.0/255.192.0.0
no-route = 153.96.0.0/255.224.0.0
no-route = 157.0.0.0/255.192.0.0
no-route = 157.96.0.0/255.224.0.0
no-route = 157.128.0.0/255.224.0.0
no-route = 157.224.0.0/255.224.0.0
no-route = 159.224.0.0/255.224.0.0
no-route = 161.192.0.0/255.224.0.0
no-route = 162.96.0.0/255.224.0.0
no-route = 163.0.0.0/255.192.0.0
no-route = 163.96.0.0/255.224.0.0
no-route = 163.128.0.0/255.192.0.0
no-route = 163.192.0.0/255.224.0.0
no-route = 166.96.0.0/255.224.0.0
no-route = 167.128.0.0/255.192.0.0
no-route = 168.160.0.0/255.224.0.0
no-route = 169.254.0.0/255.255.0.0
no-route = 171.0.0.0/255.128.0.0
no-route = 171.192.0.0/255.224.0.0
no-route = 172.16.0.0/255.240.0.0
no-route = 175.0.0.0/255.128.0.0
no-route = 175.128.0.0/255.192.0.0
no-route = 180.64.0.0/255.192.0.0
no-route = 180.128.0.0/255.128.0.0
no-route = 182.0.0.0/255.0.0.0
no-route = 183.0.0.0/255.192.0.0
no-route = 183.64.0.0/255.224.0.0
no-route = 183.128.0.0/255.128.0.0
no-route = 192.0.0.0/255.255.255.0
no-route = 192.0.2.0/255.255.255.0
no-route = 192.88.99.0/255.255.255.0
no-route = 192.96.0.0/255.224.0.0
no-route = 192.160.0.0/255.248.0.0
no-route = 192.168.0.0/255.255.0.0
no-route = 192.169.0.0/255.255.0.0
no-route = 192.170.0.0/255.254.0.0
no-route = 192.172.0.0/255.252.0.0
no-route = 192.176.0.0/255.240.0.0
no-route = 198.18.0.0/255.254.0.0
no-route = 198.51.100.0/255.255.255.0
no-route = 202.0.0.0/255.128.0.0
no-route = 202.128.0.0/255.192.0.0
no-route = 202.192.0.0/255.224.0.0
no-route = 203.0.0.0/255.128.0.0
no-route = 203.128.0.0/255.192.0.0
no-route = 203.192.0.0/255.224.0.0
no-route = 210.0.0.0/255.192.0.0
no-route = 210.64.0.0/255.224.0.0
no-route = 210.160.0.0/255.224.0.0
no-route = 210.192.0.0/255.224.0.0
no-route = 211.64.0.0/255.192.0.0
no-route = 211.128.0.0/255.192.0.0
no-route = 218.0.0.0/255.128.0.0
no-route = 218.160.0.0/255.224.0.0
no-route = 218.192.0.0/255.192.0.0
no-route = 219.64.0.0/255.224.0.0
no-route = 219.128.0.0/255.224.0.0
no-route = 219.192.0.0/255.192.0.0
no-route = 220.96.0.0/255.224.0.0
no-route = 220.128.0.0/255.128.0.0
no-route = 221.0.0.0/255.224.0.0
no-route = 221.96.0.0/255.224.0.0
no-route = 221.128.0.0/255.128.0.0
no-route = 222.0.0.0/255.0.0.0
no-route = 223.0.0.0/255.224.0.0
no-route = 223.64.0.0/255.192.0.0
no-route = 223.128.0.0/255.128.0.0
no-route = 224.0.0.0/224.0.0.0
```
服务器证书
```
ln -s /root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain.com/domain.com.crt /etc/pki/ocserv//public/domain.com.crt
ln -s /root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain.com/domain.com.key /etc/pki/ocserv/private/domain.com.key
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
再修改登录方式为证书验证。无论哪种方式都要启用系统的IP转发功能，否则无法访问网络。
