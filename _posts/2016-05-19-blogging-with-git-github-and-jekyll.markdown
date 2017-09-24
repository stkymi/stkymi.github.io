---
layout: post
title: "Git 笔记：用 Github + Jekyll 写博客"
date:   2016-05-19 10:35:06
categories: jekyll
---

<!-- more -->

### 在 Github 创建名为 username.github.io 的库

按照 [Github Pages](https://pages.github.com) 上的说明，首先要创建一个新的库，把它命名为 username.github.io。库名的第一部分需要与用户名一致才能生效。另外，不需要勾选「Initialize this repository with a README」。

创建成功后，会跳转到一个页面（如果你勾选了「Initialize this repository with a README」，则看不到这个页面），提示把已经创建的库 clone 到本地，或者输入以下命令：

{% highlight bash %}
echo "# username.github.io" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/username/username.github.io.git
git push -u origin master
{% endhighlight %}

我们不按这些来做。为了理解这些指令，这里我们先一条一条地看看这些指令到底是什么意思：

首先，我们在**本地**创建一个同名文件夹 username.github.io （之所以这样做，是为了与在网站上的库保持一致。如果我们直接把 Github 上创建的库 clone 到本地，就是一个同名的文件夹；但我们选择不直接 clone，而是自己创建一个，目的只是学习这些指令）：

{% highlight bash %}
$ mkdir username.github.io
$ cd username.github.io
{% endhighlight %}

第一行 `echo "# username.github.io" >> README.md`。命令 [echo](http://linux.die.net/man/1/echo) 起   
「display a line of text」的作用，`>>` 则起「输出到文件」的作用。所以整行翻译成中文，就是把 `echo` 的输出（也就是引号里的「# username.github.io」），输入到 `README.md` 这个文件里。输入这行后，在当前文件夹就会出现一个名为 README 的 markdown 文件，里面有一行 ‘# username.github.io’；

第二行 `git init`，把当前文件夹 Git 初始化；

第三行，`git add README.md`，把 README.md 加到 Staged Area;

第四行，`git commit -m "first commit"`, 把 README.md 的信息写到 `.git` 里，相当于给这个文件在当前的时刻截了图，这要找到这次 commit 的哈希码，就总能返回这个时刻的状态；

第五行，我们没遇到过。先不执行这行，我们输入 `git remote -v`，会发现什么都没有，也就是说，我们的 Git 文件夹还没有绑定任何的远程库（remote repository）。现在输入第五行

{% highlight bash %}
$ git remote add origin https://github.com/username/username.github.io.git
{% endhighlight %}

再输入 `git remote -v`，则会出现

{% highlight bash %}
origin	https://github.com/username/username.github.io.git (fetch)
origin	https://github.com/username/username.github.io.git (push)
{% endhighlight %}

所以，这一行的意思是，给我们本地的 Git 文件夹登记一个远程库，然后在第六行的时候，它就知道该把文件夹里的内容推送到哪里去了。更详细一点：命令 `git remote` 等于是告诉 Git 我们要关于远程库（remote）做点什么；`add` 表明要登记一些信息；`origin` 是我们给那个远程库的名字，没有必要一定要叫 origin，只是习惯如此，可以叫任何名字；最后就是要告诉 Git 那个远程库的具体地址是什么。

如果我又创建另一个叫做 Tardis 的库，它的地址是 https://github.com/username/tardis.git，我也可以把这个地址登记到我们本地的 username.github.io 文件夹里。输入

{% highlight bash %}
$ git remote add tardis https://github.com/username/tardis.git
{% endhighlight %}

再输入 `git remote -v`，则会出现

{% highlight bash %}
origin	https://github.com/username/username.github.io.git (fetch)
origin	https://github.com/username/username.github.io.git (push)
tardis	https://github.com/username/tardis.git (fetch)
tardis	https://github.com/username/tardis.git (push)
{% endhighlight %}

第六行，`git push -u origin master`，意思就是把我们本地目录 master 分支（branch）里的文件，推送到（push）名为 origin 实际上代表作 https://github.com/username/username.github.io.git 这个地址的库。我们本地在哪个分支，就会推送到远程库的哪个分支。再直白一点，命令 `git push` 是告诉 Git，我要推送当前文件夹里的东西到某个地方了，origin 指向我们要把内容推向的那个远程库，后面的 master 则是想要推向远程库的、**本地的**分支。假设，我们想要把本地一个名为 timeVortex 的分支里的文件，推送到一个名为 tardis 的远程库（无论我们当前到底是处在哪个分支），那么我们就输入 `git push tardis timeVortex`，那样子，timeVortex 分支里的文件就会被推送到 tardis 库的一个同样叫做 timeVortex 的分支上。那么，如果希望把本地 master 分支上的文件推送到远程库里的 doctor 分支上，又该怎样做呢？可以输入 `git push origin master:doctor`。冒号前面是本地的分支，冒号后面是远程的分支。

以上只是讲解,现在，我们可以把 xxx.github.io 文件夹**删掉**，再进行下一步。

### 选择模板

再次重新从零出发。这一次从选模板开始。

本文余下部分用 Scribble 为例。

### 把博客托管到 Github Pages

首先，我们把 Scribble 这个库 clone 到本地：

{% highlight bash %}
$ git clone https://github.com/username/scribble.git
{% endhighlight %}

把名为 scribble 的文件夹改名为 username.github.io (不必要，理由跟之前一样，只是为了比较好找)

{% highlight bash %}
$ mv scribble username.github.io
{% endhighlight %}

然后我们可以进入现在叫 `username.github.io` 的文件夹里:

{% highlight bash %}
$ cd username.github.io
{% endhighlight %}
	
一般而言，克隆了别人的模板，第一件事要做的就是修改 _config.yml 里的个人信息。

只要修改过文件，我们就需要重复 `git add` 和 `git commit` 这两步：

{% highlight bash %}
$ git add . 
$ git commit -m 'modified _config.yml'
{% endhighlight %}

`git add .` 里的一点，指把当前目录所有修改过的文件都加到 Staged Area 去。

前面我们说过 `git remote -v`。因为我们直接 clone 了别人的库，所以 clone 下来的文件夹里，已经登记了模板作者的远程库信息：

{% highlight bash %}
$ git remote -v
origin	https://github.com/muan/scribble.git (fetch)
origin	https://github.com/muan/scribble.git (push)
{% endhighlight %}

我们要把 origin 的地址改成我们之前创建的 username.github.io 库的地址：

{% highlight bash %}
$ git remote set-url origin https://github.com/username/username.github.io.git
$ git remote -v
origin	https://github.com/username/username.github.io.git (fetch)
origin	https://github.com/username/username.github.io.git (push)
{% endhighlight %}


最后输入

{% highlight bash %}
$ git push -u origin master 
{% endhighlight %}

如果没有绑定 SSH key，一般会要求输入用户名和密码。



