---
layout: post
title: Things I Wish I Were Told About Angular.js
---

Recently I have worked on a project using Angular.js. As of writing this post, it's a medium sized app (~10 modules, ~20 controllers, ~5 services and ~10 directives) with quite decent test coverage. When I look back, I find myself learning much more about Angular.js than before. It's not a smooth ride: I've gone through lots of refactor and rewrite. And there are lots of things I wish I were told before I started to work on Angular.js

**Heads up! The post is based on Angular.js stable branch (1.0.x). Some of the content might not apply if you choose to live on the edge (1.1.x).**

## About Learning Curve

Angular.js has a very different learning curve from Backbone.js. Backbone.js has a steep learning curve up front: to write a simplest app, you need to know pretty much everything about it. And after you have nailed it, there is pretty much nothing other than some common building blocks and best practices.

However, Angular.js is very different. The initial barrier to get started is very low: after going through the examples on homepage, you should be good to go. You don't need to understand all core concepts like module, service, dependency injection, scope. Controller and some directives can get you started. And the documentation for quick start is good enough.

The problem is when you dive into Angular.js and start to write some serious app, the learning curve suddenly becomes very steep and its documentations are either incomplete or cumbersome. It took quite some time to get over this stage and started to benefit from all these learning. This is the first thing I wish I were told - so that I would not feel frustrated when I had all these problems.

## Understand Modules Before You Start

Angular.js does not force you to use its module system. So when I started, I had a huge CoffeeScript file with lots of controllers with no segregation. It soon went out of control: it's almost impossible to navigate the file and you have to load this huge file for every page even if you are using only one of the controllers.

So I had to stop and refactor my code. I first split controllers into different modules based on different pages and then extract some common filters and services. Then I made sure all modules had its dependencies set up correctly and components are successfully injected. Of course, you also need to understand *dependency injection* (DI): a concept that is quite popular in Java and C# world but not in dynamic languages. Fortunately (or some of you may think it's unfortunate), I've done quite some Java and [C#](http://ruoyusun.com/2013/03/10/6-months-with-c-sharp.html), it did not took me very long to understand DI in Angular.js - although it did not feel natural to use DI in JavaScript for the first time. When you get used to DI, you will love the concept, it makes your code less coupled and easier to maintain.

So if you do not want to go through the refactor, learn and plan your modules before you start.

## When Your Controllers Get Bigger, Learn Scope

I am sure you use `$scope` at the very beginning, but probably not understand it. Well, you don't need to. You can treat it as magic. However, here are some very common questions you might have encountered:

Why my view will not get updated?

	function SomeCtrl($scope) {
		setTimeout(function() {
			$scope.message = "I am here!";
		}, 1000);
	}



Why `ng-model="touched"` does not work as expected?

	function SomeCtrl($scope) {
		$scope.items = [{ value: 2 }];
		$scope.touched = false;
	}



	<ul>
		<li ng-repeat="item in items">
			<input type="text" ng-model="item.value">
			<input type="checkbox" ng-model="touched">
		</li>	
	</ul>



These all have something to do with `$scope`. But more importantly, when your controllers get bigger and bigger, it's time to break it into sub controllers, and inheritance is closely related to `$scope`. You need to understand how `$scope` works:

1. How the magical data bindings is related to `$scope.$watch()` and `$scope.$apply()`;
2. How the `$scope` is shared between a controller and its child;
3. When Angular.js will implicitly create a new `$scope`;
4. How the events (`$scope.$on`, `$scope.$emit`, `$scope.$broadcast` work as a communication among scopes)

FYI, the answer to the two questions above are: for 1, you need `$scope.$apply()` and for 2, Angular.js creates a scope implicitly in `ng-repeat`.

## When You Manipulate DOM in Controller, Write Directives

Angular.js advocates separation of controller and view (DOM). But there are cases when you might need access to DOM in controllers. A typical example is jQuery plugins. Although jQuery plugins seem a little bit "outdated", the ecosystem is not going away time soon. However, it's really a bad idea to use jQuery plugin directly in your controller: you are making your controller impossible to test and you can hardly reuse any logic.

The best way to deal with it is writing your own directive. I am sure you've used directives, but you might not think you can write one. In fact, directives are one of the most powerful concepts in Angular.js. Think of it as reusable components: not only for jQuery plugins, but also for any of your (sub) controllers that get used in multiple places. It's kinda like shadow DOM. And the best part is, your controller does not need to know the existence of directives: communications are achieved through scope sharing and `$scope` events.

However, the documentation on directives is probably one of the *worst* Angular.js documentations. It's full of jargon, with very few examples and lots of abstract explanations without any context. So don't freak out if you cannot understand it. The way I get over this is to start writing directives, even without fully understanding the documentations. When I reach a stage where I don't know what to do, I go back to the documentation or Google around. After finishing wrapping a jQuery plugin into my own directive, I master most of the concepts. Learning by doing is probably the most efficient way when there is no good documentation.

For any of you who look for a word of wisdom, here is my one sentence summary: put control logic in directive controller, and DOM logic in link function; scope sharing is the glue.

## Angular.js Router Might Not Be What You Expected

This is one of the gotchas that takes me some time of figure out. Angular.js router has a very specific usage. It's not like Backbone.js router where it monitors the `location.hash` and call a function when the route matches. Angular.js works like server-side router: it is built to work with `ng-view`. When a route is hit, a template is loaded and injected into `ng-view` and some controller can be instantiated. So if you don't like this behavior, you have to roll out your own service (wrap some existing library into a Angular.js service).

This is not a generic tips, rather, it is very specific. However, I wasted lots of time on this so I just put it here and it might save you a few hours.

## About Testing

Since Angular.js advocates separation of view and controller. Writing unit tests on controller is very easy: no DOM is needed at all. Again, the purpose of unit test is not to guarantee that your app runs without a problem - that's the purpose of integration tests. Unit tests are used to quickly locate the buggy code when your integration tests fail. The only unit test that requires some DOM manipulation is directive.

Because there is dependency injection, when writing your controller unit tests, you can easily use a mock for one of the dependencies, which makes assertions much easier. A typically way to do this is use a mock when instantiate your controller. Or since Angular.js DI uses *last win rule*, you can rebind (override) your dependency by registering a fake implementation in the tests.

Angular.js also have a E2E test, which might not be easy to setup and run, especially your page is rendered by some server-side language first before JavaScript takes control. After spending quite some time on setting this up, I eventually gave up and fell back using Selenium tests as a higher level of integration tests.

Although I have been using Angular.js a lot lately, I cannot call myself an expert. In fact, one of the reason I write this post is also to learn: I will appreciated it if you can correct my mistakes or misunderstandings. Please [Discuss on HN](https://news.ycombinator.com/item?id=5770733)