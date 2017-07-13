---
layout : post
category : strong
tags : ubuntu
premalink: pretty
---
在ubuntu下更新软件的时候,经常会遇到网络连接失败的情况,其实并不是服务器的问题,而是GFW在捣鬼,所以,我们不得不给apt-get加上一个代理,使其更新的时候可以通过代理访问.

### HTTP代理###
1.在<strong>/etc/apt/apt.conf</strong>里面添加如下代码:

<pre>Acquire::http::Proxy "http://yourproxyaddress:proxyport";</pre>

再进行apt-get操作的时候就会通过代理访问目标服务器.

2.设置临时代理

切换root用户,在bash中输入:
<pre>export http_proxy=http://yourproxyaddress:proxyport</pre>

如果要使用<strong>sudo apt-get</strong>的话,你需要在<strong>/etc/sudoers</strong>里面添加:
<pre>Defaults env_keep = "http_proxy https_proxy ftp_proxy"</pre>

这样会在你sudo运行命令的时候保持原来的代理,不会因为切换了root用户而改变环境变量.

3.设置永久代理

在<strong>.bashrc</strong>文件里面添加如下代码:

<pre>http_proxy=http://yourproxyaddress:proxyport
export http_proxy</pre>

保存文件,并重新打开终端.

PS:如果http代理需要认证的话,可以将原来的代码替换为:

<pre>http_proxy=http://username:password@yourproxyaddress:proxyport</pre>

### SOCKS代理###
如果你没用http代理,而是用的 shadowsocks 或者 ssh -D 这样的代理,那么你就需要一个转发的软件了,因为apt-get默认是不支持socks代理的,尽管网上有些人将http替换为了socks,那样也是不管用的.

但是有种方法可以让 apt-get 走 socks 代理的流量, tsocks.

首先你要安装tsocks.

    sudo apt-get install tsocks

然后在<strong>/etc/tsocks.cong</strong>的最下面设置好socks代理的服务器以及端口号.

然后就可以使用代理了,具体操作如下:

    $ sudo -s
    # tsocks apt-get update
    # tsocks apt-get dist-upgrade
    # exit
    $

或者这样:

    $ sudo -s
    # . tsocks -on
    # apt-get update
    # apt-get dist-upgrade
    # . tsocks -off # not really necessary, given the exit
    # exit
    $

或者更简单点:

    $ sudo tsocks apt-get update
    $ sudo tsocks apt-get dist-upgrade

参考资料:
http://askubuntu.com/questions/35223/syntax-for-socks-proxy-in-apt-conf

https://help.ubuntu.com/community/AptGet/Howto#Part:Setting up apt-get to use a http-proxy