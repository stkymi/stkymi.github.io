---
layout: post
title: To Developers Who First Use Bootstrap (Two)
---

It's been a long time since my last post. I have to apologize for the delay as I had a little health condition. Luckily I had a draft of the second half so I can continue the writing despite of the interruption.

##Customize it the Right Way

It's acceptable to use "default" Bootstrap for internal projects or weekend project where good user experience is more important than looking good. However, for external projects or startups, using Bootstrap without any customization is a little unprofessional: looking good is as important as user experience. The former is to attract users and the latter is to keep users. So customization is very important. Bootstrap has two parts: the colors and scaffolds are easy to customize but some more high level styles (gradients, positions and margins) are not.

If you are gonna customize it, do it the right way. Download the `.less` source file. Usually, you should only change `variable.less`. But `variable.less` does not provide everything you need - sometimes you want to override some default styles. Per my experience, it's better to create your new style class and use Bootstrap class as [mixins](http://lesscss.org/#-mixins) rather than directly override the Bootstrap default. Most of the Bootstrap styles have proper "namespace" - they will not override the browser default (i.e. they require you to add special class). However, some of the Bootstrap styles are more intrusive, for example, forms will apply to any form elements without adding any class. So customizing this part is more difficult: you have to patch the relevant `.less` file.

The bottom line is: if you are going to customize Bootstrap, use a CSS preprocessor. Less is Bootstrap's choice. Lots of people prefer Sass. There's [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass) and you might like [Zurb Foundation](http://foundation.zurb.com/), which is written in Sass. From my point of view, I see more similarity in Sass and Less than differences. And most of the differences won't really affect you.

##A Good UI is More Important Than Allowing User Customization

This is not specific to Bootstrap, but is more of a general UI tips. Developers seem to like the idea of allowing user to customize the interface: dragging components around, changing color scheme, altering the layouts. Yes, it's good, and user will like it. But the fact is, it takes long time to make and it's not a must-have. Users would like to customize the interface but they can also settle for a good default interface.

The sad thing about UI is your users never agree with you, or with each other. Let's face it, you will never make everyone happy - even though you allow user to customize the interface. And you don't need to satisfy everyone. You just need 1/3 of the users to like your UI, another 1/3 **not** to hate your UI and you can leave the rest 1/3 behind.

##Responsive is a Compromise

Bootstrap 2 has a fair responsive feature (Bootstrap 3 will enhance in this aspect). However, that does not mean it will solve all your mobile problem. In fact, responsive design itself is a compromise. If you mobile is very important to you, you probably need native mobile apps, which will definitely provides best user experience. If mobile is part of your core strategy but you cannot afford to build native apps for each platform, having a dedicated mobile version is a better choice. When mobile is not your core product, but you kind of need your app to look acceptable on mobile, responsive design fits you.

##You Need (can be) a Designer After All

Bootstrap cannot replace designers. In fact, a designer can make better use of Bootstrap and make it less "Bootstrap-like". Bootstrap is not the final solution to your design problem, it is to bootstrap your product design. Design and development are quite different, but that does not mean a developer cannot be a designer. Well, it's difficult for a developer to be a "great" designer, but you can be a mediocre one. You do not need to make beautiful illustration or realistic icons. All you need is to make a "comfortable" design. Although the bar has been lowered, it is still challenging.

What I find useful is to look at good examples. Find several designs that you really like and study it. As you know, good artists copy; great artists steal. You should "steal" the essence of the good design. Why it is good? What makes it comfortable? An overview is important but good designs usually have attentions to details. Make sure you do not overlook these details.

Well, this is all I have to say, as always, please discuss on [HN](https://news.ycombinator.com/item?id=5725477).