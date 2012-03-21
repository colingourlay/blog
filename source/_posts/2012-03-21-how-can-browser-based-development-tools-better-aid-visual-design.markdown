---
layout: post
title: "How can browser-based development tools better aid visual design?"
date: 2012-03-21 23:34
comments: true
categories: [JavaScript, Browsers, Web Design]
---

As a developer, I (unsurprisingly) spend a lot of my time in the browser. Up until last year, I did the bulk of my development work in Mozilla Firefox, thanks to the invaluable debugging tools Firebug provided. These days I use Google Chrome [Canary](http://tools.google.com/dlpage/chromesxs) because I find their Web Developer Tools much more comprehensive and better suited to my development style (although Mozilla are [working on some](http://www.blueskyonmars.com/2012/02/14/firefox-2012-roadmap-for-developer-tools/) [interesting developer tools](http://www.andismith.com/blog/2012/02/firefox-developer-tools/) for Firefox).

I've always considered myself a better developer than a designer, but I'm learning more about design (not just the visual aspects) all the time. For the remainder of this post, when I say _design_, I'm referring to visual design. The current crop of browser-based development tools allow me to inspect and modify my page's CSS; some go further and overlay page with boxes that highlight the space that elements take up, as well as their margins, padding, etc. These tools help me to quickly assess the visual balance of the page, and allow me to make quick adjustments in order to improve that balance. As an aid to design, that's about as much as we can get out of our browser-based tools. But I'm greedy. I want more...

<!-- more -->

For example, I often find myself in a situation where I have nested elements, both of which have borders with rounded corners. The mistake I used to make was to give both elements the same value for `border-radius`, when what I really want is a larger radius on the outer element, and a smaller radius on the inner element, resulting in a nice even 'track' between the two borders. I've illustrated what I'm on about here:

{% img http://dl.dropbox.com/u/1545229/Websites/Blog/border_radius_before_after.png %}

There's currently a couple of ways to achieve this:

1. Get out your calculator and work out the inner radius based on the outer radius and the distance between the two borders. That's usually too much effort for most people, so you're more likely to...
2. Take a guess at the inner radius, load the page in your browser, then adjust the radius using your developer tools until it looks close enough.

But would you settle for _close enough_ if you had a visual aid that can help you get it _just_ right? Wouldn't it be great if you could use something like this:

{% jsfiddle pxjXT result,html,css,js light 460px %}

If you hover over *Example A*, you'll see some semi-transparent circles that follow the `border-radius` at each corner. Initially, the borders on the outer and inner elements have the same radius, which leaves an uneven track. Because you now have an overlay, you would be able to adjust the inner element's radius until one circle sits perfectly in the center of the other. You can then be confident that your inner radius is correct. To see the finished effect, click inside *Example A*. You can toggle between the two states by clicking repeatedly. *Example B* is a more complicated example because every corner has a different radius, but it works in the same way.

This is just a prototype I put together to demonstrate the concept. The code isn't the important part, although you can inspect it by switching tabs in the header of the jsFiddle. You can also view the demo [by itself](http://jsfiddle.net/collypops/pxjXT/embedded/result/).

## Possibilities ##

Some recent CSS grid frameworks are now [shipping with JS plugins that overlay a grid](http://goldengridsystem.com/) for you to check the alignment of your elements. We're seeing simple hacks that load up the same page in a bunch of `iframe`s - set to the dimensions of popular mobile devices - so that you can test your responsive layouts. Imagine what other concepts could be explored:

* Displaying the aspect ratios of images, thumbnails &amp; and other elements
* Painting every unique `font-size` or `font-family` a different colour in order to spot inconsistency in typography
* Adjusting your layout conform to conventions such as the Golden Ratio

There are already a few design-focused plugins for browsers, such as [MeasureIt](https://addons.mozilla.org/en-US/firefox/addon/measureit/), which raises the question that maybe what I'm proposing should be a set of plugins. However, I think there are good reasons to include this sort of functionality with the browser's built-in developer tools. It means that developers can learn and apply good design principles, and are less likely to lose some of the finer design details when translating PSDs (or other image-based designs) to HTML &amp; CSS. Also, by making these tools more useful to designers, protoypes could be built faster in the browser, and they would gain a better understanding of the capabilities and limitations of CSS, and end up producing far more realistic concepts.

I'd really like to hear your thoughts on the ideas I've explored here. Am I crazy? Or is there something worth pursuing here?