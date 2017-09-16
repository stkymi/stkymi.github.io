---
layout: post
title: To Developers Who First Use Bootstrap (One)
---

I started as a designer back in the time when the design of a web page (or app) were usually done by "professional" designers in wireframe, then in Photoshop and was sliced up nicely before it was sent to "front-end developers" who fought with cross-browser compatibility and CSS hacks. Now, this process is much simplified. You can start with some wireframes (pencil drawings is good enough) and then quickly code them in front-end frameworks (Bootstrap, Foundation, etc). Tweaking the design is as simple as changing some HTML/CSS, rather than asking designers to tweak their Photoshop, re-slice everything and adjust code while struggling with new compatibility issues.

Thanks to front-end frameworks, lots of my developers friends can actually make a complete web application all by themselves, without begging for help from designers. Actually, Bootstrap is not the first front-end frameworks. There were lots of grid frameworks before that. Actually the idea of grid was first borrowed from traditional printing design. But Bootstrap is full-stack - taking care of common UI elements like buttons, menus, labels and design choices like colors, gradients and margins. All you need to do is glue them up and perhaps customize it, which Bootstrap cannot help you a lot. I observe lots of my developer friends who have no experience in design have made some bad decisions while using Bootstrap. So here are some tips for developers who try to make use of Bootstrap.

## Stick to Bootstrap Guidelines

Bootstrap has a pretty good guidelines on how you should use it - all of its stylings are named with its intended usage. `btn btn-primary` should be used for primary action button. `alert alert-error` should be used for displaying error messages to users. Some developers use `label` as buttons because they prefer that style. Some use `alert` for normal contents because they like the color and border. These are not good ideas: they will confuse users. Bootstrap choose the specific style for button because users usually relate that to a button. If you don't like the default styles, the proper way to do it is to customize it, not misuse other elements.

## Make Use of Grid

Grid is an important guideline of Bootstrap. Lots of developers overlook this. I've seen a register page with 940px wide container with full-width navigation bar but login form is 400px wide and left-aligned, leaving lots of empty spaces on the right which makes the page look out of balance. There are two problems here. First, since there is a grid system, stick to it. Instead of making up your own layout width, use the one provided by the grid. In this case, use `span6` for register form. Second, try hard to make your content take up the whole grid. So instead of leaving the `span6` on the right blank, fill it up with something like "The Benefits of Becoming A Member" or "Community Code of Conduct", in text or in pictures.

## White Space is Important

What? Didn't you just mention that don't leave a large empty space? Yes, but `empty space != white space`. White space, or some might call it negative space, is not some space where you leave empty because you have nothing to put there. It is carefully designed to make the page more readable and delightful. An example is the margin between sections and paragraphs. Developers always want to consolidate all features into a small area so that users will never miss any of them. Unless your app is used by professionals (like Photoshop, IDEs), it will intimidate your users. You should consider only showing the most commonly used feature and use white space as a proper division of different functional areas. Bootstrap has an acceptable default margin for elements like `h1 h2 h3 p`, just make sure when you make up your own layout, use white space wisely.

## Use a Proper Color Scheme

Color scheme is probably the first thing you want to customize your Bootstrap. It will make your web app look less Bootstrap-like. But when you customize your color scheme, make sure you do it right. It is generally advised to use five or less colors on your web page. It is also safer to start with white or light background. An easier way to start is to choose one color as your "theme color" and choose another one as the "complement color". You definitely do not want your web page to look like rainbow unless you know what you are doing.

## Typography is an Design Element

Unlike color, typography is probably the last thing people usually customize. It is usually ignored by developers who try to make the app look less bootstrap-like. However, typography is an important design element. Bootstrap comes with "[web-safe font](http://en.wikipedia.org/wiki/Web_typography#Web-safe_fonts)": Helvetica/Arial, but web font is rather common nowadays. There are lots of tools for you to figure out a better font stack. Google Web Font has a decent collection of free web font and allows you to figure out which one goes well with another.

These are some tips that can help you to avoid common pitfalls when a developer with little experience of design tries to make a web app UI using Bootstrap. They are very basic - they will not make you a design master. It is more of a checklist to mark the red flags of your design. In fact, if you have lots of experience in design, you can follow none of these tips but still make a great design. But for developers who first try doing UI using Bootstrap, it is always better to start easy and safe.

This is only part one of the list, I promise I will finish the other half in a follow up post. As usual, you can discuss on [HN](https://news.ycombinator.com/item?id=5546861).