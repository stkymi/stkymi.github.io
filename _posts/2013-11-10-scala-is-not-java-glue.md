---
layout: post
title: Scala is Not Java Glue
---


Scala is enjoying all the hype now and it definitely deserves it. The language runs on one of the most mature and battle-proof language VM, has a expressive and concise syntax, provides a pragmatic mix of object oriented programming and functional programming and has a company and an active community behind it.

Targeting at JVM is a smart move, that means all of the developers that write software running on JVM can start writing Scala. And it is widely agreed that Scala is much less verbose and easier to work with than Java. So lots of people start to use Scala in the existing Java code base, starting with tests or some self-contained components. I've been involved in some project like this for a few months and my experience is: Scala is not a good glue language for Java - it's better when you write it from ground up.

One of the first problem is, although Scala can work with Java rather seamlessly, it can be verbose sometimes, diminishing some of the virtue. Collection conversion is a typical example. If the code is not structured well, you might have to do the conversion back and forth. Class and Type case is another potential trouble you have to deal with constantly. In comparison, [Groovy](http://groovy.codehaus.org/) provides real "seamless" integration with Java: you change the file extension of an existing Java code and it most likely will work (you might want to remove the semicolon). There are lots of syntax sugar that works with native Java collections.

Second problem is its learning curve. Scala is not easy to learn: the language has a huge feature set. It's not Java + Syntax sugar - in fact, it can be quite different from Java. Getting Java developers to start working with Scala can take some time and become quite depressed. The potential complexity of Java and Scala interpolation makes things even worse. [Xtend](http://www.eclipse.org/xtend/) or Groovy seems much easier to learn for Java developers as they are more similar to Java and has a smaller language feature set.

Third problem is the IDE support. Scala's IDE support is much less mature than Java. I personally quite like IntelliJ IDEA for Java. However, the Scala support is not yet very usable. Java development relies a lot on IDE, as a glue langauge, you would want a decent IDE support. IntelliJ IDEA's basic language support for Scala is slow and incomplete, partially because of the complexity of the language. Sometimes I have to wait for a few seconds before IDE can understand the code. And the refactor support is broken. Typesafe provides a "official" Scala IDE plugin for Eclipse. I never use it but have heard similar complaint. Also, Scala related tools lack IDE support, like sbt, which is almost the Scala standard building tool. This makes it a difficult choice: you want sbt which is more suitable for Scala or Ant (Maven) which works better with your IDE?

To sum up, I am not saying Scala is bad. It's just I don't think Scala is a good choice for Java glue language. In fact, if you work on a pure Scala project, most of the problems above can be avoided. If you do need a glue langauge for Java, Groovy or Xtend might work better for you.

Discuss on [HN](https://news.ycombinator.com/item?id=6706463)
