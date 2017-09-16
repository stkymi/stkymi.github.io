---
layout: post
title: My First Day with Go
---

I've always wanted to try out Go, ever since it came out. But I was always busy with other stuff so I keep procrastinating. Until recently I decided to put off everything else, put on [some](http://www.youtube.com/watch?v=QUQsqBqxoR4) [nice](http://www.youtube.com/watch?v=RBumgq5yVrA) [music](http://www.youtube.com/watch?v=47dtFZ8CFo8) and spend a day (actually only about 5 hours) with Go. So I record some of my first thoughts - I am sure when I look back after I spend more time with Go, I will regret some of the stuff I write here. But these thoughts can be useful for other Go beginners and people who want to design a new language.

## Come & Get It

The first thing is to get Go installed. I am trying Go on Windows (don't ask me why) so I get the MSI installer, double click and done. The process is very easy and quick - no painful set ups, no weird errors. Until when I try to install a package, which I will talk about later. I am happy that Windows is treated as first class citizen by Go. There are lots of languages that treat Windows as second class citizen. I am not saying Windows is a good platform, but it is widely used out there. And by not supporting this platform, a language can lose a large user base.

## Opinionated Way Of Organizing

Go is quite strict on how I should organize my code. I need `src/pkg/bin` structure and other library code is placed with my own code in `src`. Also, tests are placed side-by-side to my code. To be honest, I struggled a bit when I first tried to set up the project structure. I even tried to tweak the structure according to my preference. Eventually I gave in. After all, when I learn a new language, I almost always follow its own way of organizing things (Ruby, Python, Node, etc), I can give in again. As they say, when in Rome, do as the Romans do.

## Packages From Source Control

Go directly install package from its source control. So I sensed that I might need to install Git if I want to install packages from Github. But I tried without Git anyway - nope, didn't work. So I installed Git, tried again - nope, still didn't work. This time it complained about Mercurial (hg). Turns out the [package](https://github.com/hoisie/web) I try to install depends on a package that uses Mercurial as version control. So I also need Mercurial. What if that package needs another package that uses SVN? Do I need to installed all version control systems (supported by Go)?

It seems less friendly compare to my experience with other languages, for example, npm for Node, gem for Ruby. When I try to install I package, I just use `npm install` or `gem install` and it's done, no question asked (Well, not entirely true for Ruby packages with native extensions, then you need to install Devkit on Windows or Build Tools on Unix, but it's the problem with packages, not the package system).

## Finding a Good IDE

I've been searching for a good IDE after I wrote a hello world web app. Not because I cannot survive without an IDE, but I think Go is very IDE friendly. I think IDE support is always *a language feature*: With static type, strict namespace and file organisation, IDE support should be one of the best. Unfortunately, I cannot seem to find a good IDE. There is [liteide](https://code.google.com/p/liteide/), which offers some Go-specific support. But an IDE must be a good code editor in the first place. and liteide's code editing is very primitive. Then I tried [GoSublime](https://github.com/DisposaBoy/GoSublime), with quite a lot of tweaks, it can provide a good experience, but still doesn't feel *integrated*. I haven't tried any Go plugins for IntelliJ, my favorite IDE (I am a heavy user of RubyMine, PyCharm, PHPStorm and IDEA), but there does not seem to be an official support.

## The Name Matters

I never thought the name of the language matters. If Ruby was named "Justin Bieber", I would still like the language. But Go seems to be a bad choice of name - it's to generic! As I mentioned, Go does not have a central package repository. So every time I need to look up a package, I need to Google around. And whenever I type in `go <things i want to do>` or `go <features i need>`, the result is always mixed with lots of irrelevant stuff. I have to further tweak my search or use `golang` as the keyword. The problem is `golang` is not used by everybody so I can miss some result. This problem is not limited to searching a package - it applies to any search.

Since I've only tried Go for a limited of time, I decide not to write about the language itself. But I did go through the tutorial on golang.org. There are some quirks of its syntax and features. But I need time to digest it and look at it in an objective way. Although I write quite a few negative things here, my overall experience with Go is quite positive. The language has some interesting designs and I feel like spending more time and digging into the language. If you have any comment, [please post it on HN](http://news.ycombinator.com/item?id=6306919).