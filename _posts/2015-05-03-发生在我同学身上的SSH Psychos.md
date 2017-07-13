---
layout : post
category : code
tags : daily
premalink: pretty
---
今天下午bigtang过来说他的VPS也被lock了，DigitalOcean的staff说是有ddos攻击，第一反应是vps沦为了肉鸡，第二反应是，还真是被日了。

说起来都有点搞笑，一个搞安全的，自己的vps竟然也会被日，23333.

连接上VNC，登陆进去之后，发现本地目录多了一个 **i** 文件夹，**ps -aux** 会发现一些别的命令一直在运行（当时没有截图留念，真是可惜了），

首先做的是查看登录日志，发现在Apr 30这一天有几台主机一直在暴力登陆本机，ip段为43.255.190.0/23，香港的，一开始是以为这是几台肉鸡，被用来ssh爆破的，但是问题远远没有这么简单。

/var/log/btmp 文件是用来记录ssh登陆的，bigtang那个竟然有38M，我真是醉了，服务器的auth.log也不小，在四月三十号这天，ssh爆破进行了一天。。。唉，有个僵尸网络就是爽。。。大约爆破了十一个小时后，坏人成功了。

### 首先说一下这个木马会干什么:

1.创建开机启动脚本，放在/etc/init.d/目录下，文件名随机

2.创建周期脚本，每小时运行一次，/etc/cron.hourly/gcc.sh

3.创建一个可执行的so文件，放在 /lib/libudev.so

### 那么这个木马的恶心之处在于什么呢？

文件不死，因为脚本一直在内存中运行，会不断检测自身是否存在于 /lib/libudev.so 这个文件中，如果不存在,则创建一个并且销毁存在于其他地方的自身。这一步完成之后，这个.so文件还会创建一个开机启动脚本并且设置为开机启动，然后他就会写入shell到gcc.sh,也就是/etc/cron.hourly/gcc.sh。

这就完了？不，远远没有，他还会自动检测其他的后门、病毒、和木马，并且删除掉他们，以确保自身的安全性和隐蔽性，最重要的是对于主机的完全控制权。
然后听从来自某台主机的命令，在某个时刻统一发起DDOS攻击。。。

我粗略的扫了一下 43.255.190.0/23 这个网段，大约开了72台，并且！每台主机都开着22端口和6006端口！那么，6006莫非就是他的后门通信端口？

更为具体的说明在这：

[Threat Spotlight: SSHPsychos](http://blogs.cisco.com/security/talos/sshpsychos)

[Group Uses over 300,000 Unique Passwords in SSH Log-In Brute-Force Attacks](http://news.softpedia.com/news/Group-Uses-Over-300-000-Unique-Passwords-in-SSH-Log-In-Brute-Force-Attacks-478094.shtml)

[把精神病关进黑洞](http://www.aqniu.com/news/7303.html)

[看Level 3与思科如何联手除掉僵尸网络](http://netsecurity.51cto.com/art/201504/472078.htm)

也许还会有更为严重的后果，但是我只能看得到这么多了，坐等糖果逆向牛把它搞定。他说要反艹他们来着，23333，但是首先得有一份好的字典啊～

这是上面提供的那一部分[密码字典](../file/ssh-passwd.txt)，我在这里一并奉上，如果你有什么好的字典，记得在下面留言撒～

## 最重要的来了！怎样摆脱这个后门呢？

既然我们删除不掉它，那怎样才能摆脱它呢？

其实很简单，利用权限。

    chmod 444 /lib/libudev.so

这一条命令执行完成后，删除掉 /etc/cron.hourly/gcc.sh 
    
    rm /etc/cron.hourly/gcc.sh 
    
并且查看/etc/init.d/目录下最近修改的文件，一本来说文件名会是一串乱码，记录这个名字，然后取消其开机启动的link

    update-rc.d -f xxxxxx remove

更新之后删除这个文件。重启。然后再删除掉 /lib/libudev.so 文件。

搞定。

### 最后的最后

如果有人想玩一下这个后门，可以在虚拟机里面搞一搞，文件上传至[这儿](../file/virus.tar.gz)