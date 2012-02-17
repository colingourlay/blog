---
layout: post
title: "Safely using .ready() before including jQuery"
date: 2012-02-17 23:15
comments: true
categories: JavaScript jQuery DOM
---

Earlier today, I read [Stop paying your jQuery tax](http://samsaffron.com/archive/2012/02/17/stop-paying-your-jquery-tax), an excellent article by [Sam Saffron](http://samsaffron.com/) which explains why it's a great idea to move all of your external JavaScripts to the bottom of the HTML document (just before the closing `body` tag). Near the end of the article, he proposes a method which allows you to continue to use jQuery's DOM ready method anywhere in your document, even though you've moved jQuery itself to the bottom. His method is essentially this:

* In the `head`, include a script that:
  1. Defines an array
  2. Creates a _fake_ `$` function that pushes its argument to the array
* In the `body`, just after you include jQuery, include a script that:
  1. Uses jQuery to loop over the array's contents
  2. ...and calls the _real_ `$` function, passing in the argument.

I decided to have a play around with the code examples Sam gave and I realised that it only caters for one of jQuery's possible ways of binding to DOM ready:

``` js
$(handler) // Where `handler` is the function to bind
```

jQuery also allows the following:

``` js
$(document).ready(handler)

$().ready(handler) // although this isn't recommended

$(document).bind("ready", handler)
```

## Solution ##

With this in mind, I attempted to build upon Sam's concept, but come up with a solution that covers all four possibilities. Here's what I came up with:

``` html
<!-- In <head> -->
<script>(function(w,u){w.readyQ=[];w.bindReadyQ=[];w.$=w.jQuery=function(f){function p(x,y){if(x=="ready"){w.bindReadyQ.push(y);}else{w.readyQ.push(x);}}if(f===document||f===u){return{ready:p,bind:p}}else{p(f)}}})(window)</script>

<!-- In <body> (just after jQuery) -->
<script>(function($){$.each(readyQ,function(i,f){$(f)});$.each(bindReadyQ,function(i,f){$(document).bind("ready",f)})})(jQuery)</script>

```

OK, that looks like an unreadable mess, so I'll expand it out (with nicer variable/function names) and take you through it. Expanding the `head` script, we have:

``` js
(function (w) {

	// Define two queues for handlers
	w.readyQ = [];
	w.bindReadyQ = [];

	// Define the fake jQuery function to capture handlers
	w.$ = w.jQuery = function (handler) {

		// Push a handler into the correct queue
		function pushToQ(x, y) {
			if (x == "ready") {
				w.bindReadyQ.push(y);
			} else {
				w.readyQ.push(x);
			}
		}

		if (handler === document || handler === undefined) {
			// Queue $(document).ready(handler), $().ready(handler)
			// and $(document).bind("ready", handler) by returning
			// an object with alias methods for pushToQ
			return {
				ready: pushToQ,
				bind: pushToQ
			};
		} else {
			// Queue $(handler)
			pushToQ(handler);
		}
	}

})(window);
```

If you look at jQuery's `.ready()` [method documentation](http://api.jquery.com/ready/), it explains that if handlers bound to DOM ready using the `.bind()` function are actually triggered _after_ all other handlers have been triggered. This is the reason we have two queues - to respect that behaviour.

Expanding the `body` (just after jQuery) script, we have:

``` js
(function ($) {

	$.each(readyQ, function (index, handler) {
		$(handler);
	});

	$.each(bindReadyQ, function (index, handler) {
		$(document).bind("ready", handler);
	});
	
})(jQuery);
```

In exactly the same way as Sam's example, we use jQuery's `.each()` method to properly bind all of our queued handlers to DOM ready, but because `$(document).bind("ready", handler)` may have been called earlier, we bind these too in the correct way.

## Example ##

Here's a quick example of how to use the scripts, followed by the console output it produces.

``` html Example
<!DOCTYPE html>
<html>
	<head>
		<title>Example</title>
		<script>(function(w,u){w.readyQ=[];w.bindReadyQ=[];w.$=w.jQuery=function(f){function p(x,y){if(x==="ready"){w.bindReadyQ.push(y);}else{w.readyQ.push(x);}}if(f===document||f===u){return{ready:p,bind:p}}else{p(f)}}})(window)</script>
	</head>
	<body>
		<script>
			$(document).bind("ready", function () {
				console.log("Example D: $(document).bind(\"ready\", handler);"
			)});
			$(document).ready(function () {
				console.log("Example A: $(document).ready(handler);")
			});
			$().ready(function () {
				console.log("Example B: $().ready(handler);")
			});
			$(function(){
				console.log("Example C: $(handler);")
			});
		</script>
		<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
		<script>(function($){$.each(readyQ,function(i,f){$(f)});$.each(bindReadyQ,function(i,f){$(document).bind("ready",f)})})(jQuery)</script>
	</body>
</html>
```

``` js Example console output
Example A: $(document).ready(handler);
Example B: $().ready(handler);
Example C: $(handler);
Example D: $(document).bind("ready", handler);
```

Note that even though *Example D* is the first example, it uses `$(document).bind("ready", handler)`, so it is queued separately, and is executed after the other three examples. It behaves exactly as jQuery intends.

I hope you find this useful. Please leave suggestions (or point out errors) in the comments below.