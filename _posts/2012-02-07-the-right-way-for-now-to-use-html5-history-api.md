---
layout: post
title: The right way (for now) to use HTML5 History API (?)
tags:
- Technology
status: publish
type: post
published: true
meta:
  _edit_last: '1'
  _syntaxhighlighter_encoded: '1'
  dsq_thread_id: '567428572'
---

First off, the astute reader will note the question mark in the title above.  The disclaimer on this article is that this method is right **for me** and in my opinion seems to be more complete than some of the other workarounds I've seen on the internet.  Your mileage may vary, and in fact, you may just want to go download [history.js][1] right now and not mess with any of this.  I wanted to try to do it completely within the bounds of the native HTML5 APIs and the is the best solution I was able to come up with.

In my tests, this works with Chrome 16, FF 10, Safari 5 (all on the PC) and in fact, this was the ONLY way I was able to get consistent results across all those browsers.  IE10 should support history, but I only had IE9 on this machine, so I don't know. The reason that this was the only way I could get it working is because of an inconsistency between Webkit and Firefox.  In Webkit browsers, the onpopstate event fires on the first page load. I believe this is the wrong behavior (and [this issue][2] seems to support my belief).  In Firefox, the onpopstate event only fires (correctly) on subsequent page loads. The problem comes in when trying to do something consistent across both browsers on initial AND subsequent page loads without having the events step on top of each other. Ok, so with all that behind us, here's the code (**heavily** commented with explanations):

(Oh yeah, I'm also using [Modernizr ][3]to detect if the browser supports the history API -- if you're not using Modernizr in your project, you're seriously missing out.)

<script src="https://gist.github.com/jslaybaugh/5001887.js"></script>  

So, there you have it.  If you've never done anything with HTML5 History, that probably looks like a hot mess.  (If you want a good intro to HTML5 history, [start here][4].)  But read the comments and plug in your own data/scenario and I think you'll see its actually fairly straight forward and versatile for any scenario.  I've used this in two separate applications now with great success regardless of page load time or the complexity of the scripts involved.

If you're looking to do history updates, hopefully this helps you avoid banging your head on the desk for hours like I did. 

---

__UPDATE 6/18/2012__ - Just use [History.js][1]. Once you get the hang of it, its much easier and much more cross-browser compatible.

 [1]: https://github.com/balupton/History.js/
 [2]: http://code.google.com/p/chromium/issues/detail?id=63040
 [3]: http://www.modernizr.com/
 [4]: https://developer.mozilla.org/en/DOM/Manipulating_the_browser_history 