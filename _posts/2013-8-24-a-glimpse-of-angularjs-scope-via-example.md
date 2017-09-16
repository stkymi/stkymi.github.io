---
layout: post
title: A Glimpse of Angular.js $scope via Example
---

Angular.js is nice. I especially feel so after I started to write Backbone.js again - I had to maintain a project written in Backbone.js. Every time I write Backbone.js, I cannot help imagining how simple and easy the code *and test* will be if it is written in Angular.js.

However, Angular.js does have its own complexities. One of the most confusing part is its `$scope` life cycle. Let me use a real-life example. A few days ago, I was building this feature: when the user clicks a button, an input box appears and the focus is on the input box.

If you do this in plain jQuery, the JavaScript code looks like this (full implementation on [JSFiddle](http://jsfiddle.net/MysrN/1/)):

    $(function(){
        $('#show-menu').click(function() {
            $menu = $('#menu');
            $menu.show();
            $menu.find('input.to-focus').focus();
        });
    });
{: .prettyprint .lang-js}

The code is simple and straightforward.

In Angular.js, [you are not supposed to manipulate DOM in controller](http://ruoyusun.com/2013/05/25/things-i-wish-i-were-told-about-angular-js.html). Instead, you should write directive. Your code will look like this ([JSFiddle](http://jsfiddle.net/R2RzE/4/)):

    <div ng-controller="MenuCtrl" ng-app="App">
        <button ng-click="showMenu = true">Show Menu</button>
        <div ng-show="showMenu">
            <input type="text" focus-if="showMenu">
        </div>
    </div>
{: .prettyprint .lang-html}

    app = angular.module('App', []);
    app.controller('MenuCtrl', function () {});
    app.directive('focusIf', [function () {
        return function focusIf(scope, element, attr) {
            scope.$watch(attr.focusIf, function (newVal) {
                if (newVal) {
                    element[0].focus();
                    // You can write element.focus() if jQuery is available
                }
            });
        }
    }]);
{: .prettyprint .lang-js}

We write our own directive called "focusIf" and will trigger a focus event whenever the expression is true. It's simple and elegant, until you realize it does not work.

After some debug, you can find out that `element[0].focus()` is called. But why it does not work? Then you spend more time and realize the input box is not on the page yet - Of course you cannot focus on a `display: none` input box. Now your aim is simple: trigger the `focus` event after the DOM is updated.

To help you understand how to achieve this, let me pull out the Angular.js runtime diagram (from official document)

![Angular.js Runtime Concepts](http://docs.angularjs.org/img/guide/concepts-runtime.png)

This is what happens when the user clicks the button:

1. `$scope.showMenu` is set to `true`, Angular.js calls `$apply()` and enters `$digest` loop.
2. `$evalAsync` queue is processed. Since this queue is empty in this case, nothing is done.
3. `$watch` list is processed. We have two watchers: `ngShow` and our own `focusIf`. The order of execution is not really guaranteed (although there is an order). In this case, the empirical evidence suggests that `focusIf` is executed before `ngShow`
4. Since the `$digest` loop can cause further changes, the loop will continue until everything is processed (i.e. $evalAsync queue is empty and watchers find no more changes).
5. Angular.js loop exits, browser renders the DOM.

To make sure `element[0].focus()` will work, we need to make sure it is executed after `ngShow` watcher. The recommended way to do this is `$evalAsync` ([JSFiddle](http://jsfiddle.net/R2RzE/6/)):

    app = angular.module('App', []);
    app.controller('MenuCtrl', function () {});
    app.directive('focusIf', [function () {
        return function focusIf(scope, element, attr) {
            scope.$watch(attr.focusIf, function (newVal) {
                if (newVal) {
                    scope.$evalAsync(function() {
                        element[0].focus();
                    });
                }
            });
        }
    }]);
{: .prettyprint .lang-js}

This is the new flow:

1. `$scope.showMenu` is set to `true`, Angular.js calls `$apply()` and enters `$digest` loop.
2. `$evalAsync` queue is processed. Since this queue is empty in this case, nothing is done.
3. `$watch` list is processed. `focusIf` is executed, pushing `element[0].focus()` to the `$evalAsync` queue, then `ngShow` is executed (again ,the order of watcher execution is not really guaranteed).
4. The loop checks the exit condition: 1) Can watchers find any more changes? Nope. 2) Is $evalAsync queue empty? **Nope**
5. `$evalAsync` queue is processed. `element[0].focus()` is executed in this iteration.
6. `$watch` list is processed. There is no more watcher to process.
7. Angular.js loop exits, browser renders the DOM.

With this, we make sure `element[0].focus()` is executed after `ngShow`, the `focus` works!

However, if you've googled around on this problem, several posts recommend using `$timeout` to solve it ([JSFiddle](http://jsfiddle.net/R2RzE/7/)):

    app = angular.module('App', []);
    app.controller('MenuCtrl', function () {});
    app.directive('focusIf', ['$timeout', function ($timeout) {
        return function focusIf(scope, element, attr) {
            scope.$watch(attr.focusIf, function (newVal) {
                if (newVal) {
                    $timeout(function() {
                        element[0].focus();
                    });
                }
            });
        }
    }]);
{: .prettyprint .lang-js}

It will also work, but is not really recommended. This works because Javascript is single-threaded. All the asynchronous events are pushed into an event queue when triggered. Javascript will execute them one by one. `$timeout(Fn)` is equivalent to `setTimeout(Fn, 0)`, which tells the browser to execute `Fn` *as soon as possible* by pushing it into the event queue right away. This does not mean the execution will start *now* as event queue might be processing other code.

1. `$scope.showMenu` is set to `true`, Angular.js calls `$apply()` and enters `$digest` loop.
2. `$evalAsync` queue is processed. Since this queue is empty in this case, nothing is done.
3. `$watch` list is processed. `focusIf` is executed, pushing `element[0].focus()` to the *browsr event queue* (the big loop), then `ngShow` is executed (again ,the order of watcher execution is not really guaranteed).
4. The loop checks the exit condition: 1) Can watchers find any more changes? Nope 2) Is $evalAsync queue empty? **Yes**
5. Angular.js loop exits, browser renders the DOM.
6. Browser goes on processing the next event in the queue, which is `element[0].focus()` (As we use `$timeout`, which is provided by Angular.js and invokes `$apply`, therefore the `$digest` loop will be invoked but nothing is done since $evalAsync is empty and no changes are detected by watchers. We can replace $timeout with browser's `window.setTimeout` and it will also work. However, if the code we executed has impact to the Angular.js world, for example, trigger a watcher, using `window.setTimeout` will not achieve this since it does not enter Angular.js's `$digest` loop. You need to call `$scope.$apply()` explicitly, which is essentially what `$timeout` does).
7. The input box is focused and the processing is done, browser moves on.

The main difference of `$timeout(Fn)` is that it does not uses Angular.js's own `$digest` loop. Instead, it uses browser's native event loop.  The code will executed after the DOM is rendered, which is outside the `$digest` loop. It is essentially a hack that makes use of Javascript runtime (it's not specific to Angular.js). Since Angular.js has it own "proper" way to handle the scheduling. It is recommended to stick to that.

This example is quite simple, yet it illustrates the whole life cycle of Angular.js `$digest` loop and also in the context of Javascript event loop. If you find this intimidating, don't worry, 90% of the time you will be dealing with the bright side of Angular.js: easy data-binding, easy code reuse via directive and easy testing - Angular.js magic just works. However, as the application grows bigger and bigger, you do have to dive into the "dark" side of Angular.js from time to time. The more often you dive into such cases, the more you will understand about Angular.js.

In case I made any mistakes, please [discuss on HN](https://news.ycombinator.com/item?id=6269077).