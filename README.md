# Project: Website Optimization

**__NOTE:__ 'gh-pages' is the default branch and represents the work that I did to complete this project. 'master' is the unchanged branch forked from Udacity.**

## Running the Code
* Download the files or clone the repository.
* Open index.html in a browser to see part 1 of the project.
* Part 1 is also hosted on Github at [http://kpwagner.github.io/frontend-nanodegree-mobile-portfolio/](http://kpwagner.github.io/frontend-nanodegree-mobile-portfolio/) You can see the Page Speed results by using this url with the [Page Speed Insights Tool](https://developers.google.com/speed/pagespeed/insights/?url=http%3A%2F%2Fkpwagner.github.io%2Ffrontend-nanodegree-mobile-portfolio%2F&tab=mobile).
* Open views/pizza.html in a browser to see part 2 of the project.

## Notes for Part 1 (index.html):
* The Google Fonts link for Open Sans in the header was one of the major detriments to the Page Speed score. I eliminated the reliance on Google Fonts by not including the link. Since the instructions didn't specifically say the page must render in Open Sans, this was the simplest solution. The CSS still specifies Open Sans as the first preference in case the user has Open Sans as a system font; otherwise the style will fallback to sans-serif.
* Image optimizations (performed manually with Photoshop) were the other most significant improvement in Page Speed score.
* After creating seperate CSS files for the base style (style.css), mobile (maxwidth480.css), and print (print.css) with respective media-attribute links for mobile and print, I decided to just inline all of the css except for print. Since the site consists of only one page, and the CSS is relatively short, this seemed like an acceptable compromise. Eliminating Google Fonts and inlining all CSS are techniques I would avoid in practice, but given the high Page Speed requirement, they seemed necessary.

## Notes for Part 2 (pizza.html):
### Background Pizza Animations:
updatePositions in main.js was causing a forced synchronous layout by retreiving `document.body.scrollTop` and then applying style with each time through the for loop.

```Javascript
function updatePositions() {
  frame++;
  window.performance.mark("mark_start_frame");

  var items = document.querySelectorAll('.mover');
  for (var i = 0; i < items.length; i++) {
    var phase = Math.sin((document.body.scrollTop / 1250) + (i % 5));
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }

  // User Timing API to the rescue again. Seriously, it's worth learning.
  // Super easy to create custom metrics.
  window.performance.mark("mark_end_frame");
  window.performance.measure("measure_frame_duration", "mark_start_frame", "mark_end_frame");
  if (frame % 10 === 0) {
    var timesToUpdatePosition = window.performance.getEntriesByName("measure_frame_duration");
    logAverageFrame(timesToUpdatePosition);
  }
}
```

To eliminate the forced synchronous layout, I move the `scrollTop` query out of the for loop.

```Javascript
function updatePositions() {
  frame++;
  window.performance.mark("mark_start_frame");

  var items = document.querySelectorAll('.mover');
  var scrollTop = document.body.scrollTop;

  for (var i = 0; i < items.length; i++) {
    var phase = Math.sin((scrollTop / 1250) + (i % 5));
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }

  // User Timing API to the rescue again. Seriously, it's worth learning.
  // Super easy to create custom metrics.
  window.performance.mark("mark_end_frame");
  window.performance.measure("measure_frame_duration", "mark_start_frame", "mark_end_frame");
  if (frame % 10 === 0) {
    var timesToUpdatePosition = window.performance.getEntriesByName("measure_frame_duration");
    logAverageFrame(timesToUpdatePosition);
  }
}
```

The *mover* class elements (background pizza images) were not on seperate layers, so they were causing a lot of unnecessary painting. To cut down on the paint time, I applied `will-change: transform` to the *mover* class.

```CSS
.mover {
  position: fixed;
  width: 256px;
  z-index: -1;
  will-change: transform;
}
```

I also reduced the number of background pizzas from 200 to 24. 200 is more than needed for pratical purposes; we only need enough to fill the background of the page. 24 may not be the perfect number, but it was enough to fill my screen while reducing unnecessary background elements.

Additionally, I created a seperate pizza png for the background (pizza_resize.png) since the background pizzas are always *small* size. I didn't see much change in performance by using a smaller image, but the paint profiler indicated it needed less memory with the smaller image.

```Javascript
document.addEventListener('DOMContentLoaded', function() {
  var cols = 8;
  var s = 256;
  for (var i = 0; i < 24; i++) {
    var elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza_resize.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    document.querySelector("#movingPizzas1").appendChild(elem);
  }
  updatePositions();
});
```

### Resizing Random Pizza Images

I found two problems with the `changePizzaSizes` function that were causing the size change to take too much time. First, the `querySelectorAll` method was being called each time through the for loop; this was not necessary. Second, the function was creating forced synchronous layout by querying `offsetWidth` of pizza elements (images) then applying style to those elements (setting their width).

```Javascript
function changePizzaSizes(size) {
  for (var i = 0; i < document.querySelectorAll(".randomPizzaContainer").length; i++) {
    var dx = determineDx(document.querySelectorAll(".randomPizzaContainer")[i], size);
    var newwidth = (document.querySelectorAll(".randomPizzaContainer")[i].offsetWidth + dx) + 'px';
    document.querySelectorAll(".randomPizzaContainer")[i].style.width = newwidth;
  }
}
```

To eliminate the unnecessary `querySelectorAll` calls, I created a new variable `randomPizzas` and set it equal to the array from `querySelectorAll`. Then to eliminate the forced synchronous layout, I moved the variable `dx` and `newwidth` out of the for loop so they are only set once before any style is applied. Note, that in order to do this, I had to set `dx` and `newidth` based only on the first element in the array `randomPizzas`; this is acceptable since we want all of the pizza images to be the same size.

```Javascript
function changePizzaSizes(size) {
  var randomPizzas = document.querySelectorAll(".randomPizzaContainer");
  var dx = determineDx(randomPizzas[0], size);
  var newwidth = (randomPizzas[0].offsetWidth + dx) + 'px';
  for (var i = 0; i < randomPizzas.length; i++) {
    randomPizzas[i].style.width = newwidth;
  }
}
```










