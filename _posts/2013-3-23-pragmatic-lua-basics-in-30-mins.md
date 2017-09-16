---
layout: post
title: Pragmatic Lua Basics in 30 Minutes
---

I recently learnt Lua. By saying "recently" I actually mean two days ago. I did not learn it for any projects or work - I learnt it for fun. So I spent some time googling and playing around with [different](http://moonscript.org/) [stuff](http://luarocks.org/). Lua is probably best known as the "WoW script language", although Lua's application extends far more than that.

However, Lua is not considered as a mainstream language and it does not have hype of some languages, say Ruby, so there are not a lot of [tutorials](http://luatut.com/), especially the quick introduction of the language for programmers of other language. So I decide to write one. I have to say I am by no means a Lua expert (You probably figure that out already), so please correct me for if I make any mistake. Also, this is an pragmatic introduction to the language for *(experienced) programmers*. So I am not gonna spend time on programming basics.

## Data Types

Lua has 8 basic data types: `nil, boolean, number, string, function, table, userdata, thread`. The first five types are common across lots of languages. You can use `type(variable)` to get the type of a variable (returned in string). There is no `object` in Lua.

- Lua booleans are `true` or `false`, lower case. Only `false` and `nil` are regarded as false, other values (like `0`) are regarded as `true`.
- Lua numbers are floating points (double). There is no integer. So what looks like integer is actually represented by floating points, which is similar to Javascript
- String can be quoted in either `'single quote'` or `"double quote"`. You can create literal strings by using `[[]]`, which kind of like `"""` in Python. String is immutable in Lua, you can concat string using `..` but not `+`. When you try to do `1 + "10"`, Lua will try to cast string to number.

## Operators and Assignment

Most of them are pretty "standard" but there are several interesting features.

- Logical operators: `and or not`. You cannot use `&& || !`. Both `and` and `or` are use short-cut evaluation.
- Mathematical operators: `+ - * /`
- Relational operators: `<   >   <=  >=  ==  ~=`. The difference is `~=`, which is `!=` in most other languages.  Tables, userdata and functions are compared by reference.
- Assignment is done via `=`. An interesting feature in Lua is multiple assignment, which is like `a, b = 0, 1`. In case you write `a, b, c = 0, 1`, `c` will be `nil`. There is no syntax sugar like `a++` or `a += 1`

## Variable, Functions and Control Structure

Variables declared like this `a = 1` are global. To scope a variable, use `local a = 1`, which is similar to `var a = 1` behavior in JavaScript. In Lua, semicolons are optional, as in JavaScript. However, the convention is to omit the semicolon.

If-else syntax is like this. There is no `switch` statement.

	if cond1 then
		...
	elseif cond2 then
		...
	elseif cond3 then
		...
	else
		...
	end
{: .prettyprint .lang-lua}

Define function like this. Anonymous function is supported and since function is a data type, you can assign it to a variable, very much like JavaScript. A function can return multiple value can have [variable number of arguments](http://www.lua.org/pil/5.2.html)

	function func_name(a, b, c)
		return a, b
	end

	-- Anonymous function. And yes, I am a comment
	function (a, b, c)
		...
	end
{: .prettyprint .lang-lua}

There are three types of loop
	
	while cond do
		...
	end

	-- Repeat until the condition is true.
	repeat
		...
	until cond
	
	-- Numeric for, step can be ignored, which will default to 1. There is another generic for, which will be introduced later
	for var=from,to,step do
		...
	end
{: .prettyprint .lang-lua}

One more thing about variable scope: local variables are local to a block, which can be a function body, a control statement body or a file. There are `return` and `break`. They can only be the [last statement](http://www.lua.org/pil/4.4.html) of a block (Lua 5.2 allows `break` in the middle)

## Tables

Table is the only built-in data structure. If you have done PHP programming (unfortunately I have), it is very similar to PHP's array - it's a mixture of numerical array (list, sequence) and associative array (dictionary, map, hash). To construct a new table, you can

	local t1 = {"apple", "orange"}
	local t2 = {1="apple", 2="orange"}
	local t3 = {[1]="apple", [2]="orange"}
{: .prettyprint .lang-lua}

The above three are equivalent. Note that table index start at **1** by default, which is different from most other languages. If you construct a table with string as key, it will become an associative array

	local t2 = {one="apple", two="orange"}
	local t3 = {["one"]="apple", ["two"]="orange"}
{: .prettyprint .lang-lua}

These two are equivalent, but the second form allows keys to be any string while the first form only allows keys which are valid identifier.

To loop through a table with integer keys, you can do

	for i = 1, #t do
		print(t[i])
	end
{: .prettyprint .lang-lua}

`#` is a very handy way to get the length of a table or string.

To loop through a table with string keys, you have to do

	for key,value in ipairs(t) do
		print(key .. value)
	end
{: .prettyprint .lang-lua}

`ipairs` is a built in [iterators](http://www.lua.org/pil/7.1.html) that can help you iterate through a table.

That's it. All the important features and facts are here. The language is very lightweight and easy to learn. In my next post (I promise), I will cover some of the "advanced" topics like functions as closures, use table for object oriented programming, error handling and some built in standard libraries - these are pretty common tasks in writing "real" programs. After that, hopefully you should be able to make full use of Lua.

As always, please discuss on [HN](http://news.ycombinator.com/item?id=5427493) and correct me for any mistake.