# Project: Website Optimization

**__NOTE:__ 'gh-pages' is the default branch and represents the work that I did to complete this project. 'master' is the unchanged branch forked from Udacity.**

## Running the Code
* Download the files or clone the repository.
* Open index.html in a browser to see part 1 of the project.
* Part 1 is also host on Github at [http://kpwagner.github.io/frontend-nanodegree-mobile-portfolio/](http://kpwagner.github.io/frontend-nanodegree-mobile-portfolio/) You can see the Page Speed results by using this url with the [Page Speed Insights Tool](https://developers.google.com/speed/pagespeed/insights/?url=http%3A%2F%2Fkpwagner.github.io%2Ffrontend-nanodegree-mobile-portfolio%2F&tab=mobile).
* Open views/pizza.html in a browser to see part 2 of the project.

## Notes for Part 1 (index.html):
* The Google Fonts link for Open Sans in the header was one of the major detriments to the Page Speed score. I eliminated the reliance on Google Fonts by not including the link. Since the instructions didn't specifically say the page must render in Open Sans, this was simplest solution. The CSS still specifies Open Sans as the first preference in case the user has Open Sans as a system font; otherwise the style will fallback to sans-serif.
* After creating seperate CSS files for the base style (style.css), mobile (maxwidth480.css), and print (print.css) with respective media-attribute links for mobile and print, I decided to just inline all of the css except for print. Since the site consists of only one page, and the CSS is relatively short, this seemed like an acceptable compromise. Eliminating Google Fonts and inlining all CSS are techniques I would avoid in practice, but given the high Page Speed requirement, they seemed necessary.

## Notes for Part 2 (pizza.html):
### Background Pizza Animations:
updatePositions in main.js was causing a forced syncronous layout by retreiving document.body.scrollTop and then applying style with each time through the for loop.

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

To eliminate the forced syncronous layout, I move the scrollTop query out of the for loop.

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













