---
layout: post
title:  "Postfix | Dovecot"
date:   2017-01-16 10:35:06
categories:
---

<!--more-->

## 端口及原理知识

MUA (Mail User Agent) 可以是web-based的，例如 SquirrelMail, 也可以是桌面客户端，例如 Outlook。

一封邮件编写完成后，MUA通过SMTP协议将邮件发往一个叫做MSA (Mail Submission Agent)的服务器, 随后邮件被提交到下一站：MTA (Mail Transfer Agent)。 MSA和MTA通常是运行在不同参数配置下的相同程序，例如Postfix。

接下来MTA需要确定收件人的具体位置，这一过程通过DNS (Domain Name System)服务来完成，具体来说是一个叫做MX的DNS记录。通过DNS查询，MTA查明了邮件将发往何处。然后MTA将会通过SMTP协议将邮件发送到收件地址的MTA。

收件MTA接受的邮件下一步会被转发到 LDA (Local Delivery Agent) 或 MDA (Mail Delivery Agent)，通过它邮件将会被分发存往对应用户的邮箱里面。最后MUA通过IMAP (Internet Message Access Protocol) 或 POP3 (Post Office Protocol)协议提取邮件。

SMTP协议中，Port 25 同时用作收取其它MTA (Postfix) 发来的信件以及MUA (Outlook) 委托MTA发出信件, 即 Relay

SMTP 有定义另一个Port 587 专做 Mail Submission (MSA)，而 587 Port 的连接并不能让 MTA 接收其它 MTA 发来的信件
```
SMTP ----------– 25
SMTP     (MSA)-– 587
SMTP(tls)(MSA)-– 465
IMAP ----------– 143
IMAP(tls) -----– 993
POP3 ----------– 110
POP3(tls) -----– 995
```
### 第一篇: 设定 MTA (Mail Transfer Agent)，即: 让服务器可以接收和发送邮件

```
    apt install postfix
```
编辑 `/etc/aliases`设置别名（即转发功能）,例如将`m@****.tk`收到的邮件转发至微软的邮箱

`m: ****@***.com`

执行`postalias /etc/aliases`将此文件生成postfix可以读取的数据库文件`/etc/aliases.db`

修改Postfix的配置文件`/etc/postfix/main.cf`

```
myhostname =        描述邮件服务器的主机名称,应该有相应的A记录解析
myorigin =          发送邮件时发件人地址显示的域名
mydestination =     收件人域名在这里列出的才会被接收，而不仅仅是要求MX记录指向这里。可配置多个域名
mynetworks =        规定哪些网络IP可以使用服务器发送邮件
inet_interfaces =   默认值为all,即监听所有网络接口
inet_protocols =    网络协议 这里ipv4即可，也可以为all，则支持ipv4,ipv6
```


MTA服务器之间的TLS传输设定 /etc/postfix/main.cf
```
    smtpd_tls_security_level = may  #作为接收服务器: 支持加密但允许对方使用不加密方式传输邮件
                                     2019.6.1 如果增加这行，客户端将无法连接smtp服务器
    smtp_tls_security_level = may  #作为发送服务器: 如果对方支持加密即使用加密方式传输邮件
    # encrypt表示强制要求加密，none表示禁用
```
DKIM (DomainKeys Identified Mail) 验证: 在DNS公开一个公钥，服务器发送邮件时用私钥对邮件进行签名
```
    apt install opendkim opendkim-tools
```
创建路径（例如 /etc/opendkim/keys）生成密钥
```
    mkdir -p /etc/opendkim/keys
    opendkim-genkey -b 1024 -s mail -d example.com
    chown -R opendkim:opendkim /etc/opendkim
```
配置 /etc/opendkim.conf 
```
    Domain                  example.com
    KeyFile                 /etc/opendkim/keys/mail.private
    Selector                mail
    Socket                  inet:8891@localhost
```
这里的Domain为自己的域名，KeyFile 为域名对于的私钥，Selector 与之前生成密钥时的`-s`参数必须一致，而且会成为DNS服务商公钥存放记录的主机名前缀

打开`/etc/default/opendkim`，配置监听端口与上一步骤中的一致
```
SOCKET=inet:8891@localhost  # listen on loopback on port 8891
```
重启opendkim服务，查看端口是否被监听
```
ss -lnp | grep 8891
```

修改 /etc/postfix/main.cf 指示 Postfix 使用 DKIM 并重启Postfix
```
    # DKIM
    milter_default_action = accept
    milter_protocol = 2 
    smtpd_milters = inet:localhost:8891
    non_smtpd_milters = inet:localhost:8891
```
进入之前生成密钥的位置`/etc/opendkim/keys`，查看`.txt`文件，里面有 dkim 的公钥，将它添加为 DNS TXT 记录，类似于
```
mail._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=PpYHdE2tevfEpvL1Tk2dDYv0pF28/f5M..."
```

注意DNS服务商的记录值不能存在任何的双引号

Debug
```
    /lib/opendkim/opendkim.service.generate
    systemctl daemon-reload
    systemctl restart opendkim
    tail /var/log/mail.log | grep OpenDKIM
```
根据 SMTP 的规则，发件人的邮箱地址是可以由发信方任意声明的。发件人策略框架Sender Policy Framework (SPF) 是一种以IP地址认证电子邮件发件人身份的技术

假设邮件服务器收到了一封邮件，来自主机的 IP 是8.8.8.8，并且声称发件人为google@gmail.com。为了确认发件人不是伪造的，邮件服务器会去查询google.com的 SPF 记录。如果该域的 SPF 记录设置允许 IP 为8.8.8.8的主机发送邮件，则服务器就认为这封邮件是合法的；如果不允许，则通常会退信，或将其标记为垃圾/仿冒邮件。

在域名DNS增加一个TXT记录即可
```
    v=spf1 ip4:***.***.***.*** ~all  #允许此IP使用该域名发送邮件
或
    v=spf1 a mx ~all  # 域名自身 A 和 MX 记录指向的主机为可信主机
```
另外VPS服务商的rDNS设置为邮件域名，在DNS服务商将VPS的hostname主机名添加A记录，指向MX记录的IP地址

### 第二篇：设定 MSA (Mail Submission Agent)，即：使用SMTP协议透过服务器发送邮件
这里 SMTP 清晰一点是 SMTP Submission，即是 MUA 透过 MSA 委托 MTA 代为传送邮件 (Relay),可以使用587端口、465端口或者25端口

编辑`/etc/postfix/master.cf`，取消注释以下这行即可启用587端口
```
submission inet n       -       y       -       -       smtpd
```
而MTA与MTA之间传送邮件也是使用SMTP协议（此处必须使用25端口）

SMTP Submission 当然需要有登入认证，不然肯定會成为 Spam Mail 的 Open Relay

默认情况下，smtpd服务器只允许$mynetworks规定的IP子网使用，postfix没有账号认证功能，需要借助SASL(Simple Authentication Security Layer)组件来完成。支持的组件有Cyrus SASL 以及 Dovecot SASL

查看SMTP服务器中的SASL支持情况可用命令：
```
postconf -a
```
Cyrus SASL涉及到函数与系统相关联，不熟悉的情况下不建议配置。当我尝试卸载`libsasl2*`时，对其有依赖的众多软件均会提示被移除

~~安装Cyrus SASL~~

~~`apt install sasl2-bin libsasl2-2 libsasl2-dev libsasl2-modules`~~

Dovecot的SASL安装及配置在第四篇中记录

Postfix配置SASL以及指示SSL证书位置 (证书需要755权限）

配置文件`/etc/postfix/main.cf`
```
smtpd_sasl_auth_enable = yes
smtpd_tls_auth_only = yes   #只允许加密登录smtp
smtpd_sasl_security_options = noanonymous
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth

smtpd_tls_auth_only = yes
smtpd_tls_cert_file=/root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain/domain.crt
smtpd_tls_key_file=/root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain/domain.key
```

### ~~第三篇: 设定 MDA (Mail Delivery Agent)，即: 启用 Dovecot 管理 Virtual Mailbox~~

Postfix默认使用系统用户作为邮箱用户，并且具备LDA (Local Delivery Agent)，如果 mydestination 同時包括 domain1.com 及 domain2.com，那发出 root@domain1.com 和 root@domain2.com 都同样会寄到 root Unix user 

如果安装`dovecot-lmtpd`接管本地邮件存储，则可以创建虚拟邮箱（类似apache2的虚拟主机概念），而且可以选择启用系统用户或者虚拟用户作为邮箱用户

考虑到长时间不使用的知识点容易忘记，此处倾向于使用系统用户，并熟悉系统用户的分组与权限管理

### 第四篇: 设定 MRA (Mail Retrieval Agent)，即: 让用户从服务器收取信件

正如MTA (Postfix)提供了smtp协议，MRA (Dovecot)所提供的是POP3和IMAP协议,MUA通过这些协议与服务器取得联系

```
    apt install dovecot-core dovecot-imapd dovecot-pop3d
```
如果只需要Dovecot的SASL认证，那么安装`dovecot-core`即可


配置加密连接是否启用`/etc/dovecot/conf.d/10-auth.conf`

不加密 ~~（不仅影响POP3和IMAP服务器的连接认证，启用SASL之后的smtpd连接认证也会受此影响）~~
```
disable_plaintext_auth = no
auth_mechanisms = plain login（微软使用login认证）
```
配置ssl证书路径`/etc/dovecot/conf.d/10-ssl.conf`

```
ssl = yes
ssl_cert = </root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain/domain.crt
ssl_key = </root/.caddy/acme/acme-v02.api.letsencrypt.org/sites/domain/domain.key
```
然后关闭之前的`plain`登录认证
启用 Dovecot的SASL

配置 `/etc/dovecot/conf.d/10-master.conf`

```
# Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
    user = postfix
    group = postfix
  }
```



### 第五篇: 安装 SquirrelMail，即: 使用浏览器登录

系统应先安装apache2,再安装php，否则可能无法解析php。下载、解压缩SquirrelMail，运行`./configure`以创建初始配置文件

在SquirrelMail下载页面，下载语言包Translations，解压后运行`install`脚本。简体中文`zh_CN`默认使用`gb2312`编码,将其转换成`utf-8`编码

进入目录`locale/zh_CN/LC_MESSAGES/`转换语言文件编码：
```
    cp squirrelmail.po squirrelmail.po.bak
    iconv -f gb2312 -t utf-8 squirrelmail.po.bak >squirrelmail.po 
```
将`locale/zh_CN/setup.php` 以及 `functions/i18n.php`文件里的`gb2312`改为`utf-8`

将by2用户添加到mail组
```
    usermod -a -G mail by2
```
#### SquirrelMail 常见错误及排除
`ERROR:Connection dropped by IMAP server.` 原因：SquirrelMail找不到用户邮箱的位置。检查邮箱路径设置`mail_location = maildir:/var/mail/vhosts/%d/%n`以及文件夹`/var/mail/vhosts`的归属用户是否正确。






