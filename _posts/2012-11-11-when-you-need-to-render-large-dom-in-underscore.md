---
layout: post
title: When You Need to Render Large DOM in Underscore.js
---

For many Javascript-intensive application, front-end template is widely used. It is handy and the code is easier to maintain. In most cases, performance won't be a concern - modern browsers are very efficient in running Javascript. However, when DOM being rendered is getting larger and larger, the latency in browser becomes noticeable.

Recently, I came across a situation where I had to render few thousands rows of table in Underscore.js template. Suppose we have a large array of objects that needs to be rendered.

    var data = // very large array of objects (or in Backbone.js context, it's a very large collection)
{: .prettyprint .lang-js}

And you have Underscore.js template defined.

    <script type="text/template" id="templateSingle">
        <tr><%= field1 %></tr>
        <tr><%= field2 %></tr>
        <tr><%= field3 %></tr>
    </script>
{: .prettyprint .lang-html}

A really bad implementation, yet very common among new Underscore.js users is like this.

    var html = "";
    _.each(data, function(d) {
        html += _.template($("#templateSingle").html(), d);
    });
{: .prettyprint .lang-js}

The worst thing is the template is *complied* for every loop. This has serious performance punishment. Plus, JQuery selector is in the loop.

Actually, if you don't pass in the second parameter for `_.template`, it will return a function, which is like a complied template. In this case, you don't need to compile for every loop.

    var html = "";
    var compiled = _.template($("#templateSingle").html());
    _.each(data, function(d) {
      html += compiled(d);
    });
{: .prettyprint .lang-js}

This could have 20x performance gain, compared to the first implementation. It is reasonably fast and should fit most use cases. Another way to implement this is to put the loop in the template - looping completes after compilation.

    <script type="text/template" id="templateSingle">
        <% _.each(data, function(d) { %>
            <tr><%= field1 %></tr>
            <tr><%= field2 %></tr>
            <tr><%= field3 %></tr>
        <% }); %>
    </script>
{: .prettyprint .lang-html}

This is even faster (2x in Firefox) than the previous implementation. However, the code seems less maintainable. So it's a trade-off as to which approach is better. If the loop is not that huge, maybe maintainability is more important. However, if performance is a concern, you can move the loop into the template.

Other thoughts on optimization is to replace `_.each` with `for ... in` loop. However, empirical evidence suggests that it is not faster. Actually, it's even slower. The reason might be that `_.each` will use `array.forEach` in modern browser, which is faster than the `for ... in` loop.

I have created a jsPerf for all these test cases, which can be found [here](http://jsperf.com/underscore-js-template-rendering-large-dom).