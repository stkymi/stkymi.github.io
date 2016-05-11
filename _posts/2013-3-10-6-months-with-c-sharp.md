---
layout: post
title: 6 Months with C#
---

I have been writing C# for more than half a year now. I think it might be a good time for me to stop and think about what I have experienced with C#.

C# is not my first language - I have done lots of programming in other languages before going into C#, including C, Java, PHP, Javascript, Python, Ruby, etc. So it is relatively easy for me to pick it up - in fact, after going through some sample code, I can write some C# in a few hours. The syntax is familiar (C style), as well as the programming style (imperative).

In terms of syntax, C# is quite similar to Java - If you look at why C# is created, you should not be surprised. However, after doing lots C#, it is really painful for me to come back to Java. C# simply has much better language feature. Lambda has been with C# for a long time and it is still missing for Java. There are quite a few time savers: generators (yield), using keyword, generated getters and setters, object initializer, implicit type (var), etc.

The C# standard library is much easier to work with. Java has a decent standard library - with a detailed APIs that exposes some of the internals, which gives you full control of what you want to do. However, it can be overwhelming and make your code too verbose. C# however, provides lots of "shortcuts" that will help you achieve common tasks in simple code. For example, if you simply want to read till the end of the stream, you can use `StreamReader.ReadToEnd` in C#. While in Java, you have to write a ugly loop and read line by line. And C# has a killer feature: LINQ, which makes handling collection easy and fun. Of course, you can use Guava for Java, but without lambda, it can hardly be as good as LINQ.

I agree that tool support is a language feature. It is quite difficult for Python or Ruby to have the level of IDE that Java has. IntelliJ for Java is hard to beat. And Jetbrain (the company that makes IntelliJ) also produces Resharper (R#), a Visual Studio plugin to bring power of IntelliJ to C#. R# makes C# even easier to use. However, it is built as a Visual Studio plugin, which suffers from some buggy designs of Visual Studio (like some popup modals that does not run in background, which simply block you from doing anything). So with R#, you have powerful refactors, but it never feels as natural and integrated as IntelliJ.

Finally, it is the ecosystem. Java has a good ecosystem (let's hope Oracle will not ruin it) - lots of open source projects extending the language to every corner. Those projects are backed by some of the most successful enterprises, with inputs from great programmers all around the world. However, the ecosystem of C# has been relying on Microsoft for a long time. Although some recent trends reveal that Microsoft is trying to build a open source friendly ecosystem. In C#, it used to be the case that anything *not* from Microsoft is considered to be second class. And if Microsoft did not port some projects to C#, no one would do it. Even if some did it, no one would actually use it.

One of the factor that prevents C# to have a good ecosystem is Windows - the platform that it is mainly built for. Of course there is the Mono project but it is not a proven platform. For me, I use Mac for my personal projects and run it on a tiny Linux server. I will never consider a Windows server. First, I cannot afford the expensive license; second, my server has an ARM based CPU. For the record, I do not hate Windows, in fact, the general user experience on Windows is quite good (I am not talking about Windows 8). 

In a nutshell, from language perspective, C# is Java done right. However, because of the ecosystem and Windows, I don't see it replacing Java completely.

[Discuss on HN](http://news.ycombinator.com/item?id=5351557)