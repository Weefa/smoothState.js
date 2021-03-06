jquery.smoothState.js
===============
[![Build Status](https://travis-ci.org/miguel-perez/smoothState.js.svg?branch=master)](https://travis-ci.org/miguel-perez/smoothState.js)
[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/miguel-perez/smoothState.js?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

* [About](#about)
* [Options](#options)
* [Methods and properties](#methods-and-properties)
* [Need help?](#need-help)
* [FAQ](#faq)
* [Contribute](#contribute)
* [Built with smoothState](#built-with-smoothstate)


## About

smoothState.js is a jQuery plugin that [progressively enhances](http://www.smashingmagazine.com/2009/04/22/progressive-enhancement-what-it-is-and-how-to-use-it/) page loads to give us control over page transitions. If the [user’s browser](http://caniuse.com/#search=pushstate) does not have the needed features, it fades into the background and never runs.

### Why add page transitions at all?

Imagine, for a second, how disorienting it would be if touching a doorknob teleported you to the other side of the door. Navigating the web feels like using a teleporting doorknob. Layouts change, elements rearrange or disappear, and it takes time time for the user to adjust. Smooth transitions reduce the effort it takes for users to get settled into a new environment.

Javascript SPA frameworks, sometimes referred to as MVC frameworks, are a common way to solve this issue. These frameworks often lose the benefits of unobtrusive code. Writing unobtrusive javascript gives us more resilience to errors, performance, and accessibility.

### Hows does smoothState.js work?

smoothState.js **gives you hooks** that you can use to choreograph how the elements on your page enter and exit the page. It allows you to specify how long your animations take, and it uses the time between animations to fetch content via AJAX.

This project doesn’t dictate how you should animate things on the page. It supports CSS Animations, and allows for any popular JS animation library like [velocity.js](http://julian.com/research/velocity/).

### Design philosophy and requirements

It’s our main goal to allow us to add page transitions without having to add any logic to the backend. We keep things unobtrusive at all times.

**smoothState.js initializes on containers, not links.** Think of  containers of like small window object within the page.

1. Every url on your site should return a full layout - not just an HTML fragment
2. The smoothState container needs to have an id - a unique hook to tell us what to update on the page
3. All links and forms on the page should live within the container

These requirements makes the website resilient, since it allows us to abort and redirect the user if an error occurs. Making each link return a full page also ensures that we are creating pages with progressive enhancement in mind.

## Getting started

All we need to get started is:

1. Include a copy of jQuery and jQuery.smoothState.js on your page
2. Create a new js file and run `$(‘#main’).smoothState()`
3. Add container with an id of `#main` and include some links inside of it

```js
$(function() {
  $(‘#main’).smoothState();
});

```
By default, smoothState.js will:
* Prevent links and forms from triggering a full page load
* Update URLs and browsing history so that we do not break browsing expectations
* Use AJAX to request pages and replace the content appropriately

This default example **will not** add page transitions to your page. You’ll need to define the animations you’ll want to run using the hooks smoothState provides.

* [onBefore](#onbefore) - Runs before a page load has been activated
* [onStart](#onstart) - Runs when a page load has been activated
* [onProgress](#onprogress) - Runs if the page request is still pending and onStart has finished animating
* [onReady](#onready) - Run when requested content is ready to be injected into the page and the previous animations have finished
* [onAfter](#onafter) - Runs after content has been injected into the page and all animations are complete

## Options

smoothState provides some options that allow you to customize the functionality of the plugin. You can change the default options by passing an object into the smooth state function.

#### Options example

```js
$(function(){
  ‘use strict’;
  var options = {
        prefetch: true,
        cacheLength: 2,
        onStart: {
          duration: 250, // Duration of our animation
          render: function ($container) {
            // Add your CSS animation reversing class
            $container.addClass(‘is-exiting’);

            // Restart your animation
            smoothState.restartCSSAnimations();
          }
        },
        onReady: {
          duration: 0,
          render: function ($container, $newContent) {
            // Remove your CSS animation reversing class
            $container.removeClass(‘is-exiting’);

            // Inject the new content
            $container.html($newContent);

          }
        }
      },
      smoothState = $(‘#main’).smoothState(options).data(‘smoothState’);
});
```

### `debug`

If set to true, smoothState will log useful debug information instead of aborting. For example, instead of redirecting the user to a page on an error, it might say:

```
No element with an id of “#main” in response from “/about.html”.
```

```js
// Default
$(‘#main’).smoothState({ debug:false });
```

### `anchors`
A jQuery selector to specify to which anchors within the smoothState element we should bind.

```js
// Default
$(‘#main’).smoothState({ anchors:’a’ });
```

### `forms`

A jQuery selector to specify to which forms within the smoothState element we should bind.

```js
// Default
$(‘#main’).smoothState({ forms:’form’ });
```

### `blacklist`

A jQuery selector to specify which elements within the smoothState container we should ignore. This will apply for both forms and anchors.

```js
// Default
$(‘#main’).smoothState({ blacklist:’.no-smoothState’ });
```

### `prefetch`

There is a 200ms to 300ms delay between the time that a user hovers over a link and the time they click it. On touch screens, the delay between the `touchstart` and `touchend` is even greater. If the `prefetch` option is set to `true`, smoothState will begin to preload the contents of the url between that delay.

This technique will increase the perceived performance of your website.

```js
// Default
$(‘#main’).smoothState({ prefetch:false });
```

### `prefetchOn`

The name of the events we will listen to from anchors if we're prefetching. If you would like to throttle the prefetch, do so by firing custom events.

```js
// Default
$(‘#main’).smoothState({ prefetchOn:'mouseover touchstart' });
```

In order to add a smart throttle, you can use a library like @tristen's [hoverintent](https://github.com/tristen/hoverintent) and trigger a custom 'intent' event. Then, you just add that to the `prefetchOn` option.

```js
$(‘#main’).smoothState({ prefetchOn:'intent' });
```

Or, if you want to get a little crazy, you could go the opposite way and use @cihadturhan's [jQuery.aim](https://github.com/cihadturhan/jquery-aim) and add spider sense-like prefetching to smoothState.

```js
$(‘#main’).smoothState({ prefetchOn:'aim' });
```

### `cacheLength`

The number of pages to cache. smoothState.js will store pages in memory if cacheLength is set to anything greater than 0. This allows a user to avoid having to request pages more than once.

Stored pages will load instantaneously.

```js
// Default
$(‘#main’).smoothState({ cacheLength:0 });
```

### `loadingClass`

We apply this class to the body while a page is still loading. We won’t apply the `loadingClass` if we receive the page before the animations are complete.

```js
// Default
$(‘#main’).smoothState({ loadingClass:’is-loading’ });
```

### `alterRequest`

A function that can alter the a request’s [AJAX settings](http://api.jquery.com/jquery.ajax/#jQuery-ajax-settings) before we call it. We can use this to alter the url we’re requesting.

```js
// Default
$(‘#main’).smoothState({
  // Param `request` is an `Object` that is currently set to be used
  alterRequest: function(request) {
    // Must return and `Object` that will be used to make the request
    return request;
  }
});
```

### `onBefore`

This function runs before a page load has been activated.

```js
// Default
$(‘#main’).smoothState({
  // `$currentTarget` is a `jQuery Object` of the element, anchor or form, that triggered the load
  // `$container` is a `jQuery Object` of the the current smooth state container
  alterRequest: function($currentTarget, $container) {}
});
```

### `onStart`

This function runs when a page load has been activated. This is an ideal time to animate elements that exit the page and set up for a loading state.

```js
// Default
$(‘#main’).smoothState({
  onStart: {
    // How long this animation takes
    duration: 0,
    // A function that dictates the animations that take place
    render: function ($container) {}
  }
});
```

### `onProgress`

Run only if the page request is still pending and onStart has finished animating. This is a good place to add something like a loading indicator.

```js
// Default
$(‘#main’).smoothState({
  onProgress: {
    // How long this animation takes
    duration: 0,
    // A function that dictates the animations that take place
    render: function ($container) {}
  }
});
```

### `onReady`
* Type: `Object`
* Param: `settings` is a `Object` with two properties, `duration` and `render`

Run when requested content is ready to be injected into the page. This is when we’ll want to update the page’s content.

```js
// Default
$(‘#main’).smoothState({
  onReady: {
    duration: 0,
    render: function ($container, $newContent) {
      // `$container` is a `jQuery Object` of the the current smooth state container
      // `$newContent` is a `jQuery Object` of the HTML that should replace the existing container’s HTML.
      // Update the HTML on the page
      $container.html($newContent);
    }
  }
});
```

### `onAfter`

This function runs when content has been injected and all animations are complete. This is when we want to re-initialize any plugins on the page.

```js
// Default
$(‘#main’).smoothState({
  onAfter: function($container, $newContent) {}
});
```

## Methods and properties

smoothState provides some methods available by accessing the element's data property.

```js
// Access smoothState
var smoothState  = $(‘#main’).smoothState().data(‘smoothState’);

// Run method
smoothState.load(‘/newPage.html’);
```

The methods and properties available are:

* [href](#href)
* [load](#loadurl)
* [cache](#cache)
* [fetch](#fetchurl)
* [clear](#clearurl)
* [restartCSSAnimations](#restartcssanimations)

### `href`
Url of the content that is currently displayed.

### `load(url)`
Loads the contents of a url into our container.

### `cache`
Variable that stores pages after they are requested.

### `fetch(url)`
Fetches the contents of a url and stores it.

### `clear(url)`
Clears a given page from the cache, if no url is provided it will clear the entire cache.

### `restartCSSAnimations()`
Restarts the CSS animations of the smoothState container.

## Need help?

If you need a little help implementing smoothState there are a couple things you could do to get some support:

1. Post on stackoverflow using the [smoothState.js tag](http://stackoverflow.com/tags/smoothState.js).
2. Join the [Gitter room](https://gitter.im/miguel-perez/smoothState.js?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge) and talk to some of the contributors.
3. Contact [Miguel](http://miguel-perez.com/), he provides training and consultation services.

Please **avoid creating a Github issue** with personal support requests. We keep the tracker clear for bugs and pull requests.

## FAQ

> Help! My `$(document).ready()` plugins work fine when I refresh but break on the second page load.

smoothState provides the [onAfter](https://github.com/miguel-perez/smoothState.js#onafter) callback function that would allow you to re-run your plugins. This can be tricky if you’re unfamiliar with how AJAX works.

When you run a plugin on `$(document).ready()`, it’s going to register *only* on elements that are currently on the page. Since we’re injecting new elements every load, we need to run the plugins again, scoping it to *just* the new stuff. 

A good way to do this is to wrap your plugin initializations in a function that we call on both `$.fn.ready()` and `onAfter`. You’ll want to specify the [context](http://api.jquery.com/jQuery/#jQuery-selector-context) each time you init the plugins so that you don’t double-bind the plugins. We call this a “module execution controller”.

## Contribute

We're always looking for:

* Bug reports, especially those with a reduced test case
* Pull requests, features, spelling errors, clarifications, etc
* Ideas for enhancements
* Demos and links to sites built with smoothState.js

## Built with smoothState

Below are some really cool sites built with smoothState. Feel free to submit a pull request with your own site, or [tweet me](https://twitter.com/tayokoart) with a link.

* [Twitch Conf](http://www.twitchcon.com/) by [Alexis Gallisá](https://twitter.com/alexisg)
* [Refune](http://refune.com/) by [Victor Meyer](https://twitter.com/pudgereyem)
* [Beau Han Xu London](http://beauhanxu.com/) by [Lawrence Gosset](https://twitter.com/gosseti)

