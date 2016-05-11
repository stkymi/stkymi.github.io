---
layout: post
title: At the Right Time and for the Right Thing
---

Back in early 2011, when I was coding 17Startup.com (*it's a Crunchbase for Chinese startups*) from scratch, I was quite disappointed that I was stuck with PHP - a language that I never liked. Therefore I was exploring my other options. So I decided to give "JavaScript MVC + JSON API" a go. I had a few "good" reasons.

1. JSON output from server-side could be used for multiple "front ends" - Web, Mobile, Embedded.
2. The open API was helpful to build up a platform and be recognized as leading data provider.
3. JavsScript could provide a much better user interaction on the front end.
4. *It was the hipster tech stack at the time*

Well, to be honest, the last reason, which was probably the worst reason, was actually what dominated my decision. The others was simply what I used to reinforce myself. So here is how it went.

## One API Cannot Rule Them All

When we introduced our mobile app, which was six months later, we found out that we simply could not use the same API. The API that powered the website returned the data tailed for website - which was very different from what mobile needed. Using the same API only utilize about ten percent of the data returned on mobile, which was a waste of bandwidth. We ended up have to build a separate API for mobile only.

## Nobody Believes in Our Platform

As a startup, it's **very very very** difficult to sell yourself as a platform, especially when your customer is more established than you. So when we were trying to sell our data to those tech new websites, nobody bought it. Some were willing to try it but soon gave up as they could not see any short-term benefit. A platform was not intended for short-term, you needed to give it some time. But rarely anyone would give a startup "some time". So no one ended up using our open API.

## Our Site Does Not Need Interaction

For most of the time, our visitor only **read** on our website. Occasionally they posted some comments, "liked" some startups and that was it. There was no "heavy" user interaction at all. Even if we did not have AJAX at all, users would not mind.

## SEO Nightmare

I completely re-wrote the website after 8 months, and went with traditional server-side rendering. The most important reason is: we could not get SEO right. SEO is always a nightmare for AJAX. Most of the web **app**, which lives behind the login wall, does not really care about SEO, but we do. One of our most important traffic source was from search engine. As startups got publicity in tech news all the time, lots of people searched for them. And most of the non-web startups (e.g. mobile app, hardware, etc) does not have a website or does not do SEO at all. Therefore our website was usually the top 3 result. Google had some spider rule for crawling AJAX, which required some setup. However, the search engine giant in China, Baidu, did not crawl AJAX at all.

## Expensive to Maintain

Our old website was very expensive to maintain, both in code and daily operation. The "JavaScript MVC + JSON" approach normally takes 1.5 more work than pure server-side rendering. In server-side rendering, you get the variable from *Model*, throw it to the *View* and that's it. For our website, we needed to properly structure our JSON response. And in JavaScript, defined another set of routes, threw it to JavaScript template and then properly loaded those partials (think pyramid of doom, callback hell) in JavaScript. Another huge cost came from daily operation. For example, we spent lots of time getting Google Analytics to work but the result was a bit off, especially for complicated tracking. And we had a hard time integrating some third party widget, mostly because of script conflicts.

## Lessons Learned

I obviously had made a wrong choice at a wrong time. Even for today, building content-focused websites should still use to server-side rendering even though JavaScript MVC has matured quite a bit. SPA (Single Page Application) is really only for **web apps** living behind the login wall. Search engines still kind of works in pre-AJAX way. Unless you want to disrupt the search market, otherwise just go with it.

[Discuss on HN](https://news.ycombinator.com/item?id=7613254)