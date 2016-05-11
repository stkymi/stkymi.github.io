---
layout: post
title: Pragmatic Lua - Error Handling, OOP, Closure and Coroutine
---

This is a follow-up post of my previous Lua introduction. If you are new to Lua, you might want to read [Pragmatic Lua Basics in 30 Minutes](http://ruoyusun.com/2013/03/23/pragmatic-lua-basics-in-30-mins.html) first.

## Error Handling

Lua does not have try-catch like syntax for error handling. In fact, error handling in Lua is very minimal. It is usually done via `error()` and `pcall()`. In most other languages, you can write something like this (I use Lua syntax)

	function i_might_throw_exception()
		if something_is_wrong then
			throw Exception("Something is wrong")
		end
		...
		return something
	end

	try
		local result = i_might_throw_exception()
	catch Exception e
		print(e)
	end
{: .prettyprint .lang-lua}

Well, in real Lua, you have to write following

	function i_might_throw_exception()
		if something_is_wrong then
			error("Something is wrong")
		end
		...
		return something
	end

	local success, result = pcall(i_might_throw_exception)

	if not success then
		print(result) -- print out error message
	end

{: .prettyprint .lang-lua}

`pcall` will run the function in protected mode. The first return value is always a boolean, indicating whether the function call is successful (i.e. no errors). The return value of the original function will be returned following this boolean flag. If something is wrong, the second return value will be error message.

## Object Oriented Programming

Lua does not have built-in object type. Neither does it have class. OOP is a paradigm, you don't need `object` in the language to do OOP. In Lua, OOP is usually done via table. In other languages, you can think `object` as a structure that holds values and has methods. Since function is first-class citizen, a table in Lua can hold values and functions, which is essentially the `object`.

Some common tasks of OOP in other languages are

	class Vehicle
		public speed
		
		function constructor(initial_speed)
			this.speed = initial_speed
		end

		function speed_up(by)
			this.speed = this.speed + by
		end

		function stop()
			this.speed = 0
		end
	end

	class Car extends Vehicle
		function speed_up(by)
			if this.speed + by > 100 then
				this.speed = 100
			else
				this.speed = this.speed + by
			end
		end
	end

	local v = new Vehicle(50)
	v.speed_up(60)
	print(v.speed) -- 110
	v.stop()
	print(v.speed) -- 0

	local c = new Car(50)
	c.speed_up(60)
	print(c.speed) -- 100
	c.stop()
	print(c.speed) -- 0
{: .prettyprint .lang-lua}

In real Lua, you can write

	Vehicle = {}

	function Vehicle.new(self, initial_speed)
		local obj = { speed = initial_speed }
	  	setmetatable(obj, { __index = Vehicle })
		return obj
	end

	function Vehicle.speed_up(self, by)
		self.speed = this.speed + by
	end

	-- this is a syntax sugar, and is equivalent of writing function Vehicle.stop(self)
	function Vehicle:stop()
		self.speed = 0
	end
	
	function Car:new(initial_speed)
		local obj = { speed = initial_speed }
	  	setmetatable(obj, { __index = Vehicle })
		return obj
	end

	function Car:speed_up(by)
		if self.speed + by > 100 then
			self.speed = 100
		else
			self.speed = self.speed + by
		end
	end

	local v = Vehicle.new(50)
	v.speed_up(v, 60) -- or you can write v:speed_up(60)
	print(v.speed) -- 110
	v:stop()
	print(v.speed) -- 0

	local c = Car.new(50)
	c:speed_up(60)
	print(c.speed) -- 100
	c:stop()
	print(c.speed) -- 0
{: .prettyprint .lang-lua}

Two points to note:

- Constructtion of a new object and inheritance are achieved via `setmetatable`. metatable is a normal table with special keys that are hooked up to events. In this case, `__index` is triggered whenever the caller tries to look up some key that does not exist in the table. This works exactly like prototype-based inheritance in JavaScript.
- The syntax `Table:method(a, b, c)` is a syntax sugar. When defining a method, it is equivalent to `Table.method(self, a, b, c)`, when you call the method, it is equivalent to `table.method(table, a, b, c)`. Essentially, you are passing the table as a parameter in function.

## Closure

Closure in Lua is very similar to JavaScript, but the concept of closure is not straightforward. [This StackOverflow answer](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work) is particularly useful in understanding the concept.

	function speed_up()
		local speed = 50
		return function()
			speed = speed + 10
		end
	end

	su = speed_up
	print(su()) -- 60
	print(su()) -- 70
{: .prettyprint .lang-lua}

In the above example, when you call `su()`, you have access to `local speed`, even though you are out of scope already.


## Coroutine

Lua has built-in coroutine support. Coroutine itself is quite easy to understand. It is a block of code that can be paused and resumed. However, to actually make use of coroutine to achieve concurrency is difficult. I will only introduce Coroutine basics in Lua. If you come from Ruby world, fiber in Ruby (added in 1.9) is quite similar to coroutine. In fact, they borrow `yield` and `resume` from Lua.

	local co = coroutine.create(function ()
		coroutine.yield(1)
		coroutine.yield(2)
		coroutine.yield(3)
	end)
{: .prettyprint .lang-lua}

When you create a coroutine, it will not automatically start. In fact, it will pause at `yield`, you have to `resume` it.

	coroutine.resume(co) -- true, 1
	coroutine.resume(co) -- true, 2
	coroutine.resume(co) -- true, 3
{: .prettyprint .lang-lua}

`coroutine.resume()` first returns a boolean flag indicating whether there is any error - the coroutine body is running in protected mode. The values you yield in coroutine body is gonna be returned following the boolean flag. When the coroutine yields all values, it will be dead and when you try to resume a dead coroutine, it will return error.

	coroutine.resume(co) -- false, cannot resume dead coroutine
{: .prettyprint .lang-lua}

Coroutine works like a `thread`, however, the pausing and resuming is done in the code, not in OS level. And you cannot force a coroutine to pause at any time - coroutine only pauses at the point you have specified. Switching between coroutines costs considerably small and synchronization is explicitly achieved in your code. However, coroutine is not real multi-threading in the sense that any blocking code will block the whole program, while in multi-threading it only blocks that thread. That's why to achieve currency, you have to make your code non-blocking.

That's the end of my introduction to Lua. Again, like I said in my previous post, I am by no means a "Lua expert". So please correct me for any mistakes and join further discussions on [HN](https://news.ycombinator.com/item?id=5462519).