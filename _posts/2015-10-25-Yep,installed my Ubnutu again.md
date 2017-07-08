---
layout: post
tags : daily
premalink: pretty
title: 对，我又重装了。
---
At first I have to say what I'm writing blow is all bull shit.

For these days I reinstall my Ubuntu, this is a sad story...

The reason is that I made a mistake when I repartitioned the disk. At first I want to resize the EFI partition and the boot partition because the original EFI partition is too big for me(500M) and the boot partition is a little small(just 200M), you know it's unreasonable, so I decided to resize it.

First I made a Live-CD to to all of this, and I intended to shrink the EFI partition and then enlarge the boot partition. I backup the files in the EFI partition before I used gparted to resize the EFI partition but I suddently realized that it changed the format of the EFI partition from FAT32 to FAT16, I'm quiet unhappy of it so I decided to formart it to FAT32 again. I know it will destory all the data in this partition. But as I have backuped the files before, I'm not afraid to lost any data. 

……

然后悲剧就开始了。EFI格式化之后，把文件复制进去，然后把boot分区扩大，原本扩大完就没事了的，不知道我脑子又抽了什么疯，莫名其妙的boot也给格式化了，现在已经回想不出当时是怎么一回事了，忘掉最好。然后怎么办？重装boot呗！

由于我的根目录是经过加密的，要先解密mapper成lv才能挂载，于是乎：

````bash
mount /dev/mapper/ubuntu--vg-root /mnt
mount /dev/sda2 /mnt/boot
mount /dev/sda1 /mnt/boot/efi
````

把这三个分区挂载到系统中，然后再把运行相关的文件挂载到里面：

````bash
for i in /dev /proc /sys /run; do mount -B $i /mnt$i; done
````

-B 选项：
> The bind mounts.
 Since Linux 2.4.0 it is possible to remount part of the file hierarchy somewhere else. The call is
 	mount --bind olddir newdir
 or shortoption
 	mount -B olddir newdir
 or fstab entry is:
 	/olddir /newdir none bind

对。
最后：

````
chroot /mnt
````

这样就进入了原来的系统。有兴趣的同学可以仔细看下 chroot 这个命令。

然后悲剧继续。

由于boot分区已经清空了，所有的启动相关的文件全部丢失，没了内核。怎么办？**sudo apt-get remove linux-generic** 不管用，因为这些内核的状态全部是 *rH* `dpkg -l | grep linux-image`可以看到，*r* 表示removed，*H* 表示没安装完全，安装了一半……（废话，boot没了，当然剩下一半了……）纠结了一天，终于在吃饭之前的某个瞬间想到，我可以把丢失的文件复制回来！然后就在Livd-CD启动的系统上装了原来系统的内核们（对，就是“们”，5个！）然后把boot分区中的内容复制到原来系统的boot分区下，然后再到原来系统中，不管是remove也好，还是purge也好，反正是把那些内核卸载了，然后，重装内核。

大功告成了么？Naive!
由于源磁盘是加过密的，重装之后，按说启动时应该让输密码解密，但是，没有解密这个步骤。也不知道时丢失了那个文件，但是在fstab文件（/etc/fstab 保存着系统的文件分部，挂载什么，从哪儿挂载，UUID是啥，挂在的模式是什么）中看着是对的，现在想想有可能是cryttab文件（crypttab - static information about crypted filesystems）中的UUID出了问题，反正就是坏了。。

怎么办？

最坏的打算就是重装了，幸好周四中午的时候刚备份完（备份完之后就残废了！）。

中间我还想着增加一块SWAP分区，因为原来的时候把SWAP给删了，SSD太小，而且觉着swap实在鸡肋（虽然现在仍然这么想，但重装之后还是把swap给加上了，官方推荐3个G）。

由于加密的是pv，pv又分了一个vg，vg里面就一个lv。（你有可能不懂这是什么，PV啦，VG啦,LV啦。不懂得回去看鸟哥的私房菜，第三版第八章，我也是刚看，唉，如果之前仔细看看这一部分，就不至于重装一边系统了，厚积而薄发，厚积而薄发啊！）

想着先把原来的lv缩小，敲了一些乱七八糟的命令（大忌！如果不懂这个命令是什么，不要随便乱敲！），那天晚上看的时候不太懂，隐隐约约先把lv缩小了，但是！lv的filesize没改变！这里要注意的是缩小lv之前一定要先把lv的filesize减小，要不就会出现superblock和lv大小不一样的情况，网上搜索了很多，从09年到12年都又人遇到了这个问题，估计是和我一样手贱，乱敲命令，但是都没有解决方案，都没有解决方案！有一个中文博客，提到了这一点，说是操作顺序如果有误，就会出现这样的问题，从而导致下一步的检测无法通过，但是并没有给出解决错误的方案，无奈。我也只好死马当活马医，下面提示igore的时候忽视他，进行手动修复，大不了重装。

自动修了大约一个小时，然并卵。

好吧，我服了，重装。里面的所有配置都丢了，包括三个kali虚拟机，一个xp虚拟机，以及xp里面磨了三年的菜刀和养的肉鸡。日了狗！

下一篇谈谈干货，重装之后都做了什么，适合新手和老手阅读。