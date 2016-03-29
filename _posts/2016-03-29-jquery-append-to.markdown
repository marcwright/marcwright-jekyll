---
layout: post
title: jquery append() v. appendTo()
date:   2016-03-29 04:29:23 -0500
published: true
categories:
---

As an exercise, I started creating a [tic-tac-toe game in Codepen](http://codepen.io/marcwright/pen/NNgzbY) using Javascript and jQuery only (no HTML or CSS). The difference between `append()` and `appendTo()` is pretty straightforward, but I'd never really come across a sufficent reason to use one over the other until I started building my game. I had a interesting realization I wanted to share.

Both methods append one object to another, but the difference lies in the way you want to order the objects. Let's say that I wanted to `append()` a new `div` with the id of `container` to `body`:

~~~javascript
$('body').append("<div id='container'></div>");
~~~

In this example, we create a jQuery object from the CSS selector `body` then we `append()` a new HTML element to it. Another way to do this would be to first create the new element, assign it to a variable, then use `appendTo` to append it to the body object: 

~~~javascript
var newElement = "<div id='container'></div>";

$(newElement).appendTo('body');
~~~

Both appear to give us the same result so when should we use one over the other?

To build my gameboard, I used a `for` loop to dynamically create 9 divs. I also needed to attach `click` events to each div in order to track the moves.

For my first attempt at this, I created a jQuery object out of `$('#container')` then appended the newly created divs to it. When I clicked an individual `div` I returned 9 `console.log` messages instead of 1. I realized that somehow I had attached 9 click events to the `#container`. Then I decided to try something like this:

~~~javascript
for (i=0; i < 9; i++) {
  $("<div class='cell' id='cel" + i + "'></div>")
    .one('click', function(e){
      index = parseInt((e.target.id)[3]);
      moves[index] = (counter % 2 == 0) ? "X" : "O"; 
      counter++;
    })
    .appendTo('#container');
};
~~~

Using `appendTo` allowed me to FIRST successfully add click events to each `div` BEFORE appending to the `#container`. I learned that order matters.