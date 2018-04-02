---
layout: post
title:  "Postfix | Dovecot"
date:   2017-01-16 10:35:06
categories:
---

<!--more-->

> ### 第一篇: 设定 MTA (Mail Transfer Agent)，即: 让服务器可以接收和发送邮件

```
    apt install postfix
```
MTA服务器之间的TLS传输设定 /etc/postfix/main.cf
```
    smtpd_tls_security_level = may  #作为接收服务器: 支持加密但允许客户端使用不加密方式传输邮件
    smtp_tls_security_level = may  #作为发送服务器: 如果对方支持加密即使用加密方式传输邮件
```    
    # encrypt表示强制要求加密，none表示禁用

DKIM验证: 在DNS公开一个公钥，服务器发送邮件时用私钥进行签名

    apt install opendkim opendkim-tools

创建路径（例如 /etc/opendkim/keys）生成密钥

    mkdir -p /etc/opendkim/keys
    opendkim-genkey -b 1024 -s mail -d example.com
    chown -R opendkim:opendkim /etc/opendkim

配置 /etc/opendkim.conf 以及 /etc/default/opendkim

    Domain                  example.com
    KeyFile                 /etc/opendkim/keys/mail.private
    Selector                mail

    Socket                  inet:8891@localhost

修改 /etc/postfix/main.cf 指示 Postfix 使用 DKIM

    # DKIM
    milter_default_action = accept
    milter_protocol = 2 
    smtpd_milters = inet:localhost:8891
    non_smtpd_milters = inet:localhost:8891

Debug

    /lib/opendkim/opendkim.service.generate
    systemctl daemon-reload
    systemctl restart opendkim
    tail /var/log/mail.log | grep OpenDKIM

发件人策略框架(SPF)是一种以IP地址认证电子邮件发件人身份的技术

    v=spf1 a ip4:***.***.***.*** ~all  #允许此IP使用该域名发送邮件


> ### 第二篇: 设定 LDA (Local Delivery Agent)，即: 启用 Dovecot 管理 Virtual Mailbox

安装Dovecot的LMTP服务「Local Mail Transfer Protocol service」，以接管Postfix收到邮件后的本地存储以及投递
    apt install dovecot-core dovecot-lmtpd
    mkdir /var/mail/vhosts
    chown by2:by2  /var/mail/vhosts
修改邮件目录 /etc/dovecot/conf.d/10-mail.conf，%d代表Domain；%n代表Username 

    mail_location = maildir:/var/mail/vhosts/%d/%n
修改登入授权方式 /etc/dovecot/conf.d/10-auth.conf；禁用系统用户而改用虚拟用户

    # 注释掉
    #!include auth-system.conf.ext
    # 启用
    !include auth-static.conf.ext
虚拟用户方式的具体设定 /etc/dovecot/conf.d/auth-static.conf.ext

    # static userdb with shared uid and gid
    userdb {
    driver = static
    args = uid=vmail gid=vmail
    }
    # using password file
    passdb {
    driver = passwd-file
    args = /etc/dovecot/passwd
    }
添加虚拟用户至密码文件：此处使用Dovecot的dovead内置pw工具生成密码

    echo name@domain.com:`doveadm pw`:::::: >> /etc/dovecot/passwd

修改Dovecot lmtp设置 /etc/dovecot/conf.d/10-master.conf

    service lmtp {
      unix_listener /var/spool/postfix/private/dovecot-lmtp {
        mode = 0600
        group = postfix
        user = postfix
      }    
      # Create inet listener only if you can't use the above UNIX socket
      #inet_listener lmtp {
        # Avoid making LMTP visible for the entire internet
        #address =
        #port = 
      #}
    }
修改 /etc/postfix/main.cf，加入以下两行，并在mydestination删除域名：

    virtual_mailbox_domains = domain1.com
    virtual_transport = lmtp:unix:private/dovecot-lmtp

> ### 第三篇: 设定 MDA (Mail Delivery Agent)，即: 让用户从服务器收取信件

postfix并未带有MDA，需要安装Dovecot提供IMAP及POP3支持

    apt install dovecot-imapd dovecot-pop3d

> ### 第四篇: 安装 SquirrelMail，即: 使用浏览器登录

转换 locale/zh_CN/LC_MESSAGES/ 语言文件编码，并修改 locale/zh_CN/setup.php 以及 functions/i18n.php

    cp squirrelmail.po squirrelmail.po.bak
    iconv -f gb2312 -t utf-8 squirrelmail.po.bak >squirrelmail.po 

将by2用户添加到mail组

    usermod -a -G mail by2






