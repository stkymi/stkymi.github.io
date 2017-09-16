---
layout: post
title: My Take on CoffeeScript
---

I discovered CoffeeScript when it first came out. I thought it was just a bunch of syntactic sugar for brackets (braces and parentheses) haters. Back then, I did not write that much JavasScript and I do not hate brackets. Therefore I decided that I would not pick it up.

## Not a painless start

However, when I wrote more and more JavaScript, I was finally fed up with its verbosity and I decided to give CoffeeScript a try. After going through the "tutorial" on homepage, I started to write some code. The first thing I found annoying was that I had to go back to the "tutorial" quite a few times. The tutorial itself is neither comprehensive nor well-organized. The table of content has only one level and the order and grouping of topics seems to be completely random. For example, the topic on "if else" is near the beginning but "switch" is much later and is grouped with "try/catch". "Scoping" is at the beginning with "function binding" (fat arrow) comes much later. Topics on object and class, operators and chained comparison are far away from each other. Something important like "everything is an expression" appears in the middle, buried in the overwhelming tutorial. The tutorial is not comprehensive - I had to use instinct (or my guess) and dug into the compiled JavaScripts to see how it worked. For example, there is `or=` feature which is never documented in the tutorial.

Luckily, after the painful start, I went on fast track. I could certainly feel the boost in productivity and it solves some of the real life problems.

## Mandatory indentation is good

Personally, I don't mind mandatory indentation - I (as well as almost all programmers) will indent properly for languages with optional indentation. There are cases where different editors use different settings for indentation or you have copied some code snippet from other sources. Although the code might look correctly indented, for example, part of the code use 4 spaces and part of the code use tab  - and you editor is set to tab = 4 spaces. When other people open in their editors with different setting, it will look totally messed up. With mandatory indentation, what you see is what compiler sees. Of course, this is a relative edge case. For teams, probably you will enforce everyone to use same indentation settings in the first place. But this does happen in real life, by mistake or laziness.

## Everyone loves sugar

You might wonder, is replacing `function(data) { return doSomething(data); }` with `(data) -> doSomething(data)` a great improvement? It is, especially when you have to type in this quite frequently every day. The latter is concise, readable and natural.

On a side note, I'd rather prefer CoffeeScript compiles function definition `a = () -> something` into named function expressions `var a = function a() { something };`. Then debuggers will print out function names in stack trace, instead of "anonymous".

CoffeeScript is bundled with carefully designed syntactic sugar, aiming to simplify your daily code as well as bring joy back to code. It certainly does the job well. When I have to go back to JavaScript, I find myself miss the lovely shortcuts the most. CoffeeScript absorbs lots of good parts from Ruby and Python, with perhaps more influence from Ruby. I am glad that CoffeeScript inherits the conciseness and expressiveness from Ruby.

## A mix of functional programming and object oriented programming

JavaScript is quite functional in essence, but it borrows its syntax from Java, a rather non-functional language. That's why the language seems a little unnatural. CoffeeScripts brings more elements of functional programming to the language, which makes you feel that this is the way JavaScript should be. For example, one of the most important feature of CoffeeScript is "Everything is an expression". Which means you can do something like `isLargerThanSix = if x > 6 then yes else no`

JavaScript comes with object oriented paradigm but the inheritance mechanism - prototype-based inheritance, is a little tricky to work with, since most people are brainwashed by class-based mechanism. CoffeeScript provides a *thin* layer on top of JavaScript that makes it more familiar for people without much experience with function prototype. The complicated hierarchy of inheritance, which is common in Java and C#, is never popular with JavaScript. CoffeeScript does not over-engineer this class structure, which makes you to write CoffeeScript in JavaScript way - keep hierarchy simple.

## Full of surprises

Some people consider CoffeeScript to [have bad readability](http://ceronman.com/2012/09/17/coffeescript-less-typing-bad-readability/), while readability is a subjective term, I certainly agree that CoffeeScript has a bad *predictability* - it is full of surprises. Part of the reason is its white space sensitivity. In my opinion, the fact that compiler is tolerant and allows multiple ways to accomplish one thing also contributes to its poor predictability. Take a look at this example:

	set User, -> name: "John", silent: yes

What do you expect to be the result (suppose this is what you want)?

	set(User, function() { return {name: "John"}; }, {silent: true});



Wrong, this is what you get
	
	set(User, function() { return {name: "John",silent: true}; });



How about this?

	set User, -> 
		name: "John",
	silent: yes

Wow, what you get is a disaster

	set(User, function() { return {name: "John"}; });
	({silent: true});



Will this work?

	set User, -> 
		name: "John",
		silent: yes

Nope. How about this?
	
	set User,
		-> name: "John"
		silent: yes

Yay, it works! By the way, this will also work

	set User, -> name: "John",
	silent: yes

But this will give you the disaster case

	set User, -> name: "John"
	silent: yes

To avoid ambiguity, you probably want to write something like

	set User, (-> name: "John"), silent: yes

And the worst thing is, non of the above behavior is documented in the CoffeeScript official website, which makes it a try and error process - the least efficient way to learn. One way people usually use to avoid this is to enforce a coding standards, to make sure everyone are in the same pace and no one uses the "dark magic".

## It's just JavaScript, really?

One of the reason why CoffeeScript gets popular is that you can easily interpolate with JavaScript library - it's just JavaScript. It's true, however, CoffeeScript has its own features and there might be hidden traps if you still think in JavaScript way.

For example

	$('#li').each ->
		update @

This is very common in JavaScript. Your `update` might return `true` or `false` to indicate whether it has been updated but you don't care about the return value - you just want to update everything. Now look at the translated code

	$('#li').each(function() {
		return update(this);
	});



In CoffeeScript, the last line will be returned by default. which is fine. But in this case, your `.each` will stop (break) if the anonymous function body returns an `false`. That means, although you want to update all record, your loop will simply stop at the first record where `update` returns a false.

As you write more CoffeeScript, you will become very careful on this, especially when interpolating with 3rd party libraries. Sometimes you have to compile the code in your mind to see whether the behavior matches your expectation.

## Conclusion

Well, now I write CoffeeScript most of the time - I like the language and I never look back. In fact, I feel painful when I have to go back to JavaScript. There are certain drawbacks with CoffeeScript, but most of them are avoidable and when you write more CoffeeScript, you will experience less problems and understand more about language.

[Discuss on HN](https://news.ycombinator.com/item?id=5389054)