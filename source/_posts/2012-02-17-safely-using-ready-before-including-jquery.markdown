---
layout: post
title: "Safely using .ready() before including jQuery"
date: 2012-02-17 23:15
updated: 2012-03-02 23:15
comments: true
categories: [JavaScript, jQuery, DOM]
---

Earlier today, I read [Stop paying your jQuery tax](http://samsaffron.com/archive/2012/02/17/stop-paying-your-jquery-tax), an excellent article by [Sam Saffron](http://samsaffron.com/) which explains why it's a great idea to move all of your external JavaScripts to the bottom of the HTML document (just before the closing `body` tag). Near the end of the article, he proposes a method which allows you to continue to use jQuery's DOM ready method anywhere in your document, even though you've moved jQuery itself to the bottom. His method is essentially this:

1. In the `head`, include a script that:
    * Defines an array
    * Creates a _fake_ `$` function that pushes its argument to the array
2. In the `body`, just after you include jQuery, include a script that:
    * Uses jQuery to loop over the array's contents
    * ...and calls the _real_ `$` function, passing in the argument.

I decided to have a play around with the code examples Sam gave and I realised that it only caters for one of jQuery's possible ways of binding to DOM ready:

``` js
$(handler) // Where `handler` is the function to bind
```

<!-- more -->

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
<script>(function(w,d,u){w.readyQ=[];w.bindReadyQ=[];function p(x,y){if(x=="ready"){w.bindReadyQ.push(y);}else{w.readyQ.push(x);}};var a={ready:p,bind:p};w.$=w.jQuery=function(f){if(f===d||f===u){return a}else{p(f)}}})(window,document)</script>

<!-- In <body> (just after jQuery) -->
<script>(function($,d){$.each(readyQ,function(i,f){$(f)});$.each(bindReadyQ,function(i,f){$(d).bind("ready",f)})})(jQuery,document)</script>
```

OK, that looks like an unreadable mess, so I'll expand it out (with nicer variable/function names) and take you through it. Expanding the `head` script, we have:

``` js
(function (w, d, u) {

    // Define two queues for handlers
    w.readyQ = [];
    w.bindReadyQ = [];

    // Push a handler into the correct queue
    function pushToQ(x, y) {
        if (x == "ready") {
            w.bindReadyQ.push(y);
        } else {
            w.readyQ.push(x);
        }
    }

    // Define an alias object (for use later)
    var alias = {
        ready: pushToQ,
        bind: pushToQ
    }

    // Define the fake jQuery function to capture handlers
    w.$ = w.jQuery = function (handler) {
        if (handler === d || handler === u) {
            // Queue $(document).ready(handler), $().ready(handler)
            // and $(document).bind("ready", handler) by returning
            // an object with alias methods for pushToQ
            return alias;
        } else {
            // Queue $(handler)
            pushToQ(handler);
        }
    }

})(window, document);
```

If you look at jQuery's `.ready()` [method documentation](http://api.jquery.com/ready/), it explains that if handlers bound to DOM ready using the `.bind()` function are actually triggered _after_ all other handlers have been triggered. This is the reason we have two queues - to respect that behaviour.

Expanding the `body` (just after jQuery) script, we have:

``` js
(function ($, doc) {

    $.each(readyQ, function (index, handler) {
        $(handler);
    });

    $.each(bindReadyQ, function (index, handler) {
        $(doc).bind("ready", handler);
    });
    
})(jQuery, document);
```

In exactly the same way as Sam's example, we use jQuery's `.each()` method to properly bind all of our queued handlers to DOM ready, but because `$(document).bind("ready", handler)` may have been called earlier, we bind these too in the correct way.

## Example ##

Here's a quick example of how to use the scripts, followed by the console output it produces.

``` html Example
<!DOCTYPE html>
<html>
    <head>
        <title>Example</title>
        <script>(function(w,d,u){w.readyQ=[];w.bindReadyQ=[];function p(x,y){if(x=="ready"){w.bindReadyQ.push(y);}else{w.readyQ.push(x);}};var a={ready:p,bind:p};w.$=w.jQuery=function(f){if(f===d||f===u){return a}else{p(f)}}})(window,document)</script>
    </head>
    <body>
        <script>
            $(document).bind("ready", function () {
                console.log("Example D: $(document).bind(\"ready\", handler)");
            });
            $(document).ready(function () {
                console.log("Example A: $(document).ready(handler)");
            });
            $().ready(function () {
                console.log("Example B: $().ready(handler)");
            });
            $(function(){
                console.log("Example C: $(handler)");
            });
        </script>
        <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
        <script>(function($,d){$.each(readyQ,function(i,f){$(f)});$.each(bindReadyQ,function(i,f){$(d).bind("ready",f)})})(jQuery,document)</script>
    </body>
</html>
```

``` text Example console output
Example A: $(document).ready(handler)
Example B: $().ready(handler)
Example C: $(handler)
Example D: $(document).bind("ready", handler)
```

Note that even though *Example D* is the first example, it uses `$(document).bind("ready", handler)`, so it is queued separately, and is executed after the other three examples. It behaves exactly as jQuery intends.

I hope you find this useful. Please leave suggestions (or point out errors) in the comments below.

##### Updated on 02/03/2012: #####
In the comments, BRUNOa A suggested a couple of performance enhancing changes to my solution. As an extra enhancement, I'm also passing the `document` object as an argument into the anonymous functions. The original implementation still exists as a [gist](https://gist.github.com/1958226).
