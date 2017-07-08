---
layout : post
category : daily
tags: [build-my-own-blog]
premalink: pretty
---
I suggest U visit this site first.

[-Jekyll'home page-](http://jekyllrb.com)

1. md文件末尾不能存在空格，否则浏览器解析会出错误

2. jekyll插入图片格式 “ !\[name\](link) ”

3. md的名字中存在的"-"会被解析成空格

4. 首先要在本地搭建好jekyll环境，

<pre>
sudo apt-get install gem	
sudo gem install jekyll
sudo gem install [相关组件]
</pre>

可以搜一下 [jekyll-now](https://google.kfd.me/search?client=ubuntu&channel=fs&q=jekyll-now&ie=utf-8&oe=utf-8) 或者 [jekyll bootstrap](http://jekyllbootstrap.com/)，都有详细的教程。

大体目录：
<pre>
.
├── 404.html		\#404页面
├── about.html		\#about页面
├── archive.html	\#汇总页面
├── assets			\#css，js文件存放
│   ├── css
│   │   ├── pygment-trac.css
│   │   └──  stylesheet.css
│   └── js
│       ├── main.js
│       └── top.js
├── audio.mp3		\#自己加的背景音乐
├── _config.yml		\#配置文件
├── feed.xml		\#订阅
├── home.html		\#主页，我改成了home
├── img				\#图片文件存放目录（插入图片格式:" !\[nam\](link)"）
│   ├── bkg.png
│   ├── favicon.png
│   ├── rocket.png
│   └── s
│       └── remove_attribution_in_blogger
│           └── remove_attribution_in_blogger.png
├── _includes		\#\{ % include  "文件名" %\}，把这里面的文件包含到网页中
│   ├── analytics.html
│   ├── disqus.html
│   ├── header.html
│   └── head.html
├── index.html		\#我的主页
├── _layouts		\#模板文件存放目录
│   ├── default.html
│   ├── index.html
│   ├── page.html
│   └── post.html
├── music.html		\#背景音乐
├── _posts			\#博文目录
│   ├── code
│   │   └── 2014-12-22-linux下利用C实现输入密码返回*的功能.md
│   ├── daily
│   │   ├── 2014-12-06-my github.io blog.md
│   ├── others
│   │   ├── 2014-10-04-关于ARP欺骗.md
│   ├── problems
│   │   ├── 2014-10-24-关于ubuntu下MP3文件名乱码的问题.md
│   ├── share
│   │   ├── 2014-10-07-hydra使用方法.md
│   │   └── 2014-12-25-Guide to Vim.md
│   └── ubuntu
│       ├── 2014-12-13-你可能不知道的shell.md
│       └── 2014-12-23-Ubuntu自定义Applications.md
├── README.md
├── sitemap.html	\#sitemap
└── tag.html		\#tag-html
</pre>

美化博客在于修改css和js，框架只是框架。

我是先挑好了一个框架，包括字体，链接样式，背景样式，然后再在这个基础上进行修改。

其中碰到了很多问题，首先就是这头疼的语法：
<pre>\{\{ xxx.xxx \}\}、\{ % include xxx.html %\}、\{ % if xxx.xxx %\}\{ %end if%\}、\{ %for xxx.xxx %\}\{ end for%\}</pre>

在[这儿](http://jekyllrb.com/docs/variables/)、[这儿](http://jekyllrb.com/docs/frontmatter/)和[这儿](http://jekyllrb.com/docs/permalinks/)有详细的说明。

写的比较详细的教程：

http://blog.csdn.net/on_1y/article/details/19259435

