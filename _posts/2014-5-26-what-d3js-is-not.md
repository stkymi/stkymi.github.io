---
layout: post
title: What D3.js is Not
---

I have played with D3.js quite a bit recently. After exploring its API and building a rather complex chart, I come to realize that I have misunderstood D3 for a long time. It's not only me, after talking to my friends, they also have misconceptions regarding D3. So I've decided to write this post to clear some of the common misunderstandings.

## D3 is Not a Charting Library

When you go to D3 homepage, you can see lots of amazing charts and visualizations. But D3 is not a charting library like [Highcharts](http://www.highcharts.com/), [Chart.js](http://www.chartjs.org/) or [Google Charts](https://developers.google.com/chart/). You cannot simply pass in your dataset, specify the type of the chart you need and get a fancy chart. D3 is much lower level than that. There are charting libraries build on D3, e.g. [nvd3](http://nvd3.org/), [Rickshaw](http://code.shutterstock.com/rickshaw/)

## D3 is Not a Graphics Layer

D3 is not a graphics layer - in fact, most of its graphic power comes from SVG. D3 essentially provides a data friendly API for manipulating SVG (or even HTML, since they are all XML based markup language), it does not "draw" the graphics itself - the heavy-lifting of the representation is done by SVG.

## D3 is Not a SVG Polyfill

Unlike [Raphaël](http://raphaeljs.com/), which provides polyfill for SVG on browsers that do not support SVG. D3 manipulates SVG directly, without any abstraction layer. Your browser needs to support SVG for D3 to work properly.

## D3 Does Not Support Canvas or WebGL (Almost)

Even though some of the D3 API (like `d3.geo.path()`) can work with both SVG and Canvas, most of its API is designed for SVG. If you are looking for Canvas Library, check out [Paper.js](http://paperjs.org/), [Fabric.js](http://fabricjs.com/) and [EaselJS](http://www.createjs.com/#!/EaselJS). [Three.js](http://threejs.org/docs/) is a decent library for WebGL.

## D3 is Not Designed to Work With AngularJS

AngularJS has its own DOM manipulation API (data binding). And So does D3. To make the two work together, one of them has to take control on DOM. You can either use an AngularJS directive and pass the DOM to D3, which will do its magic, or only use the data transformation API of D3 and let AngularJS deal with DOM. However, either way, you ended up not utilizing a large part of API of either framework.

## But D3 is Awesome

After playing with D3 for a while, I would define D3 as a *data visualization* tool, in the sense that its API has two parts: *data* and *visualization*. D3 comes with lots of handy utilities for processing data (array, time series, geo data), which is quite useful by itself. Its powerful visualization API makes it easy to bind, not only data, but also its transformation to documentation.

D3 is not very easy to learn. So when you start, it's important to have an adequate expectation. Also, get your hands dirty early and work your way through.

Discuss on [HN](https://news.ycombinator.com/item?id=7944591)