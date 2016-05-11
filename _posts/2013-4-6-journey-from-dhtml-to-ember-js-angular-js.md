---
layout: post
title: Journey From DHTML To Ember.js/Angular.js
---

The number of times that I revisited JavaScript is probably the most among all the programming languages I use. And every time I revisited the language, it felt like writing a complete new language. Although the syntax stays the same, the application of the language has changed dramatically.

## DHTML era

If you start writing JavaScript early enough, I am sure DHTML is very familiar to you. This is when I first heard of JavaScript. A JavaScript "master" at that time showed me all kinds of tricks he did: toggling some content, dynamically changing font colour and adding some "cool" animations. There were lots of websites that had useful code snippets — I just copied them, few people bothered to dig into the language. And the code at that time could be difficult to understand, mostly because of the DOM API, which was tedious to work with. Most people didn't treat JavaScript as a legitimate programming language, they considered it as a quick and dirty tool that you pulled out once in a while.

## AJAX and jQuery

When AJAX first came out, I thought it was an interesting proof-of-concept, I never thought I was going to use it. After all, JavaScript was such a terrible language. Then jQuery came out (there were also prototype.js and mootools, but everyone turned to jQuery). First time I looked at the examples, I thought it was a new language. Then I realised it was just JavaScript and I thought I should take time to learn this language. This is when I learnt all the quirks of the language: functional programming, lexical scope, prototype inheritance, closures, etc. And jQuery just made me feel that [I was flying](http://xkcd.com/353/). Soon, there were lots of jQuery plugins out there: slider, modal box, animations, etc. And you simply included the script file and added a one-liner. And AJAX also became really popular: you hit some URL, got some HTML snippet (later with JSON) and updated your existing HTML document, with just a few lines of code. No need to worry about cross-browser, and you had elegant jQuery API to help you with DOM manipulation.

During this time I wrote lots of DOM coupled jQuery spaghetti code. I did not feel it was a problem until I wrote my first "web app". I ended up with several thousands lines of jQuery code in one file that coupled with DOM. There was no tests, no namespace and no structures. It could break any time and naming conflicts were common. Everytime I modified the file, I had to search the function name (with try and error) and tried not to break everything.

## Backbone.js/Knockout.js

Before Backbone.js came out, JavaScript templating system was introduced. I first read it on [John Resig's blog](http://ejohn.org/blog/javascript-micro-templating/). Before that, when JSON was returned via AJAX, people either used jQuery to update DOM or used string concatenation to build HTML from JSON. Neither way was scalable. JavaScript template solved this problem — it made JavaScript strong enough to handle the view layer.

So Backbone.js seemed to be a natural evolution. However, the first I used Backbone.js, I was very frustrated. To achieve a simple task, I had to write far more code than plain jQuery. For example, to display a list of items from JSON, you need a model, a collection, an item view, a list view and a "main" to bootstrap your app. And you can achieve the same in jQuery with a few lines of code. It was when I had my thousands lines of jQuery horror I mentioned above that I realised the value of Backbone.js — paying an upfront cost and getting maintainability in return. It was not easy to transit to Backboke.js. The most difficult thing was the way the code was organised. In jQuery, things are coupled with DOM, so it is direct and straightforward. For Backbone.js, you have to re-organise your code: put the data logic in model and UI bindings in view. 

Backbone.js is very lightweight and the code of Backbone.js itself is easy to understand. There is no magic — everything is rather explicit. Especially Backbone.js does not provide a UI binding: you are free to use your own. UI binding is like catalyst to your development. You can survive without it, but you will be way faster with it, kind of like ORM in backend web frameworks. So using Backbone.js is like writing Rails without ActiveRecord.

There is a library that is focused on UI binding - Knockout.js. It uses declarative bindings solves UI binding pain by providing a two-way binding (observable) mechanism. However, Knockout.js itself does not provide an application structure, nor does it provide a model layer. Also, it does not come with routing support. So you have to deal with organisation problem as in jQuery.

## Ember.js/Angular.js

Ember.js and Angular.js are to solve the problem of modern JavaScript application development. In essence, they combine the good parts from Backbone.js and Knockout.js - providing an application structure, view-data segregation as well as the powerful two-way binding. But they achieve this in a very different way. Ember.js is like adding a two-way binding, as well as other useful features to Backbone.js, while Angular.js is like adding application structure, modular system and dependency injection to Knockout.js. They all are designed to be full-stack framework, which makes them best for large applications in JavsScript. So if you want to choose one framework for your next project, don't ask questions like "Ember.js vs Angular, which is better?". Some people does not feel comfortable adding bindings directly in HTML, others think Ember.js is overkill. Any strong opinions you've read about these two frameworks are most likely arguing which design philisophy *they like better*.  The easiest way to decide is to try them out. You can easily find one you like more.

## Advice to new JavaScript Learners

If you just start to learn JavaScript, consider yourself lucky - you don't have to take the detour, instead, you can go for the express way. Here is some advice for you.

- Learn the language. Javascript has lots of quirks that are different from other languages. You have to *really* understand them in order to become a serious JavaScript developer.

- Learn jQuery. Trust me, you don't want to use JavaScript DOM API. Since everyone is using jQuery, it is like the *de facto* standard of DOM API. 

- If you find Ember.js/Angular.js is difficult (especially Ember.js), Backbone.js is a good way to get started. It is lightweight, explicit and easy to learn. After you understand how Backbone.js works, learning Ember.js is much easier.

- For rapid JavaScript application development, use Angular.js/Ember.js (or something that provides application structure and powerful UI binding).

[Discuss on HN](https://news.ycombinator.com/item?id=5502816)