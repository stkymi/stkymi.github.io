---
layout : post
targs : code
premalink : pretty
---

今天处理了一个Gates病毒。
病毒表面症状是，创建一个带有 `ais` 隐藏属性的 `/etc/kblockd` 文件，无法删除，并且在 `/usr/bin/dpkgd/` 目录中创建了`lsof netstat ss ps`这些二进制文件，在 `/tmp` 目录下生成 `gates.lod` 和 `moni.lod` 两个文件，在 `/etc/init.d/` 中创建 `selinux` 和 `DbSecuritySpt` ，并向 `/etc/rc.d/rc*/` 中注册，其中，`/etc/init.d/selinux` 启动的是 `/usr/bin/bsd-port/getty`，还会创建一个隐藏的 `/usr/bin/.sshd` 文件作为后门。

看上去很复杂的样子。

解决脚本如下：

```bash
#!/bin/bash
charrt -ais /etc/kblockd
#cp /dev/null /etc/kblockd
#chmod -x /etc/kblockd
rm -f /etc/kblockd

pkill .sshd
chmod -x /usr/bin/.sshd
rm -f /usr/bin/.sshd

pkill ^getty
rm -rf /usr/bin/bsd-port
rm -rf /usr/bin/dpkgd

find /etc/rc.d/ -name *selinux -delete
find /etc/init.d -name *selinux -delete
find /etc/rc.d/ -name *DbSecurity -delete
find /etc/init.d -name *DbSecurity -delete

yum erase net-tools && yum install -y net-tools
yum erase lsof && yum install -y lsof
yum install iproute
yum install procps

rm -f /tmp/moni.lod
rm -f /tmp/gates.lod
```

别人的链接：

<http://wangzan18.blog.51cto.com/8021085/1740113>

<http://beijing0414.blog.51cto.com/8612563/1725375>

## 解决木马的套路

Linux下的病毒木马之类还是很好解决的，只不过鲜有人知道方法。比如 `chattr` 和 `lsattr` 这个关于隐藏属性的两个命令。

1. 确定木马进程，找出异常的进程和开放的端口。
2. 确定木马文件，一般就是那些删除不掉的文件，这个时候要用`lsattr`查看文件的隐藏属性，去除隐藏属性之后再删除。
3. 如果删除了文件之后还会再次生成，那么肯定还有一个其他的进程在守护着这个病毒，所以，这里有个办法就是，去除文件的执行权限，或者，`cp /dev/null <木马文件>`，这样这个文件就无法执行了。
4. 然后再去查找上一层的木马进程，一般这东西就写在 `/etc/rc.local` 或者 `/etc/init.d/` 或者 `/etc/rc.d/rc*/` 这些目录下，还有就是，用户的 `~/.bashrc` 文件，这个是容易忽略的，也不知道有没有木马往这里写东西，总之，都看一下比较好。
5. 如果确认不了木马的上层文件，那么就在确认 `ps` 命令没有被替换的前提下，一个一个的查找进程，如果 `ps` 命令都被替换了，那就重新安装一下 `ps`. 然后通过 `ps -a <异常进程ID>` 来确认异常文件。
6. 最后就是检查一下开放的端口，和对应的程序，发现异常之后，通过 `chmod -x` 或者 `cp /dev/null` 的方式先取消掉文件的执行能力，然后再顺藤摸瓜的查找。
7. 重启机器，看是否还有异常。

最后，其实很多公司都不注意安全的，比如ssh默认端口不更改，root用弱口令，关闭防火墙等等。

机器常升级，补丁勤着打，防火墙要开，默认端口记得改，别用弱口令，能用堡垒机就用堡垒机，服务能简单就简单，能用容器跑，尽量就用容器跑，还要注意做好隔离。

其实这些木马都很傻逼的，无非就是这些套路。

哦对，还有一点，就是注意一下 `/etc/passwd` 这个文件，不要被人添加了别的用户，然后uid改成0，这样就不好了。