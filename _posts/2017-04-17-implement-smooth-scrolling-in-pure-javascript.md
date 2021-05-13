---
layout: post
title: "Implement smooth scrolling in pure Javascript"
date: "2017-04-17"
---

Smooth scrolling is something very cool, we all like. Most of the time, people add it to their websites via [jQuery plugins](https://github.com/cferdinandi/smooth-scroll). Although those plugins are great, they might not be a good fit if your are a guy called Jimmy who doesn't want to add jQuery to his project. If you are this guy, stay with me, I will show you how to implement smooth scrolling yourself, with vanilla Javascript.

 

## It's all about animations

Smooth scrolling, carousels, games,  and many other dynamic things we see on webpages, are all based on a powerful javascript feature: **animations**. An animation consists in moving an element from one state to another, by incrementally changing one or many of its properties during a period of time.

[caption id="attachment_352" align="aligncenter" width="534"]![start](/public/img/start.png) Starting point[/caption]

[caption id="attachment_353" align="aligncenter" width="625"]![end](/public/img/end.png) Ending point[/caption]

 

Thinking of smooth scrolling, the animation here will consist in scrolling a webpage from its current scroll position to a target position, by incrementally changing it's scroll position, during a period of time. Okay, we now know that smooth scrolling is a kind of animation, what's next ?

 

## Picking an easing function

As I said in the preceding section, to make our page scroll smoothly, we will need to incrementally change it's scroll position. But the most important thing, is the rate at which that position will change. Does the value speed up initially and then slow down ? Does the value instead start slowly and then speed up? An easing function, is what we use to calculate this rate.

[caption id="attachment_373" align="aligncenter" width="625"]![easing](/public/img/easing.png) Easing function[/caption]

Easing functions are based on mathematical equations, and we will cover them in another article.

Now, we need to choose an easing function for implementing our smooth scrolling. Let's pick the **easeInOutQuad** function from Robert Penner's very popular easing functions. You can find them all on [Github](https://github.com/jaxgeller/ez.js). The **easeInOutQuad** function will make our page scrolling fast at the beginning, and then slow as it comes to the target.
```js
//easeInOutQuad
function easing(t, b, c, d) {
  t /= d / 2;
  if (t < 1) return c / 2 * t * t + b;
  t--;
  return -c / 2 * (t * (t - 2) -1) + b;
}
```
 

## Writing the smooth scrolling function

### Computing the page current scroll position

var startPosition = window.scrollY || window.pageYOffset;

_Both of the_ `window.scrollY` _and_ `window.pageYOffset` _properties contain the number of pixels that the webpage is currently scrolled from the top._

### Computing the target element position

var targetPosition = target.getBoundingClientRect.top();

_Here we assume_ `target` is _the scroll target element_. _The_ `getBoundingClientRect`  _method called on an element, returns a  `DOMRect` object, which contains the `left`, `top`, `right`, `bottom`, `x`, `y`, `width`, `height` properties describing the border-box in pixels; the `top` property is the one we need here._

### Computing the distance between the page current scroll position and the target element position

var distance  = targetPosition - startPosition;

### Putting the blocks together
```
/*
* target is the element we want to scroll to
* duration is the scroll delay in ms
*/

function smoothScroll (target, duration) {
  var startPosition = window.scrollY || window.pageYOffset;
  var targetPosition = target.getBoundingClientRect.top();
  var distance = targetPosition - startPosition;
  var timeStart = null;

  function loop(timeCurrent) {
    if(timeStart === null) timeStart = timeCurrent;

    timeElapsed = timeCurrent - timeStart;

    next = easing(timeElapsed, startPosition, distance, duration);

    // Vertically scroll from the current scroll position to next
    window.scrollTo(0, next);

    // Rerun the animation until the delay(duration) expires.
    if(timeElapsed < duration) window.requestAnimationFrame(loop);
  }

  // Our good old easeInOutQuad function
  function easing(t, b, c, d) {
    t /= d / 2;
    if(t < 1) return c / 2 * t * * t + b;
    t--;
    return -c / 2 * (t * (t - 2) - 1) + b;
  }

  // Start running the animation
  window.requestAnimationFrame(loop);
}
```

_The_ **`window.requestAnimationFrame()`** _method tells the browser that we are going to perform an animation. It takes a callback method (_`loop`_) that will run on each update. We keep running the animation until the scrolling delay(_`duration`_) has expired._

_Each time it's called, the_ `loop` _function receives as a default argument, the current_ `timestamp`_.  Thanks to our_ `easing` _method, we are able to calculate the_ `next` _position to scroll to. Then, we use the_ `window.scrollTo()` _method to incrementally scroll._

### Binding the click event on links to the smoothScroll function
```
document.querySelector('a').onclick = function(event) {
  smoothScroll(event.target, 1000); // scroll delay here will be 1s
}
```

That's it ! We have just implemented our own smooth scrolling function in pure javascript.

Thanks for Reading !
