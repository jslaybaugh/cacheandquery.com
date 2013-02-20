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

    (function ()
    	{
    		var _firstLoad, _id = null; // the value of _id would be set by your server side code on page load if it had something to pre-load from the URL
    
    		// in this function, i'm assuming you'd use the id passed in to do some ajax call and update the page
    		var loadContent = function (id, push)
    		{
    			if (id != null)
    			{
    				// ajax call to your server to pull data
    				//
    				// ... [ your code here] ...
    				//
    				// update page with pulled data
    
    				// if browser supports history (check using Modernizr) *AND* we've passed true to push, then we pushState
    				if (Modernizr.history && push)
    					history.pushState({ id: id }, "", "/" %2B id); // update the url to something like /5 or /6 (depending upon id)
    			}
    			else
    			{
    				// load whatever the "initial" or empty page would be in your application
    			}
    		};
    
    		// in case you want to see the order in which these fire
    		//alert('code');
    
    		window.onload = function ()
    		{
    			// in case you want to see the order in which these fire
    			//alert('load');
    
    			// every browser properly fires the onload event, so in this, we set a boolean value when it is the first page load
    			// we also use replaceState to replace the current state with whatever property we currently need to display.
    			// Since this only fires on the FIRST load, we don't have to worry about messing any pushStates up, plus we ensure there's something in the
    			// history if/when we return to this page later
    
    			_firstLoad = true;
    
    			// if we have a value for _id on first load, we use REPLACEstate to make sure this is on the history stack
    			if (_id != null && Modernizr.history) history.replaceState({ id: _id }, "", "/" %2B _id); // update the url to something like /5 or /6 (depending upon _id)
    
    			// since we used replaceState, we don't want to push something on the history stack on top of it, so we load the content,
    			// but pass false to avoid it getting pushed on the history stack
    			loadContent(_id, false);
    
    			// this is the only "hacky" part of what we have to do, but this makes the _firstload = false not fire until AFTER the onpopstate fires
    			setTimeout(function () { _firstLoad = false; }, 0);
    		};
    
    		// check to see if their browser even supports history API using Modernizr
    		if (Modernizr.history)
    		{
    			window.onpopstate = function (event)
    			{
    				// in case you want to see when this event fires
    				//alert('pop');
    
    				// this is the key to everything right here.
    				// this event fires in webkit on page load AND history pops, but in firefox, it only loads on history pops
    				// so, to standardize, we use the _firstLoad value from earlier to see if this is a first page load
    				// if it is a first page load (that means we're in Webkit) and we just exit out but we set _firstLoad = false
    				// so that we can properly handle future pops, but this way, we don't get any pops on the initial page load,
    				// regardless of what any goofy browser wants to do on the inital page load.
    
    				if (_firstLoad)
    					_firstLoad = false;
    				else
    				{
    					if (event.state != null)
    						// there is something in the history state for this entry, so we go ahead and load it
    						// but we pass in false so that it doesn't write another entry over it...
    						loadContent(event.state.id, false);
    					else
    						// if there is nothing in the state (either first load or returning to a page that was a first load)
    						// then we tell it not to load the ajax, but instead just load the default content
    						loadContent(null, false);
    				}
    			};
    		}
    
    	}).call(this);
    

So, there you have it.  If you've never done anything with HTML5 History, that probably looks like a hot mess.  (If you want a good intro to HTML5 history, [start here][4].)  But read the comments and plug in your own data/scenario and I think you'll see its actually fairly straight forward and versatile for any scenario.  I've used this in two separate applications now with great success regardless of page load time or the complexity of the scripts involved.

If you're looking to do history updates, hopefully this helps you avoid banging your head on the desk for hours like I did.

- Jorin

 

**UPDATE 6/18/2012** - Just use [History.js][1]. Once you get the hang of it, its much easier and much more cross-browser compatible.

 [1]: https://github.com/balupton/History.js/
 [2]: http://code.google.com/p/chromium/issues/detail?id=63040
 [3]: http://www.modernizr.com/
 [4]: https://developer.mozilla.org/en/DOM/Manipulating_the_browser_history 