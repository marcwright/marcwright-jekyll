---
layout: post
title: Taylor Swift HTML
published: true
date:   2016-02-08 15:29:23 -0500
categories:
---
[CodePen](http://codepen.io/marcwright/pen/XdWVrG)

Both songwriting and good code are about patterns. We encapuslate our code for readability and reusability. Songs are similar. When building a track, you may reuse a snippet of audio from one verse to pop into another. When you're using Logic or Pro Tools you also want your environment to be very clean and use good naming conventions. 

With that in mind, I decided to take stab at what Taylor Swift's "Style" would look like as scaffolded in HTML5. Here are some assumptions...

####I needed a song that was very consistent and linear
This song has only 4 chords and can be easily broken down into 2 bar sections. 

####Headers will be the main section dividers
I'm gonna use headers to encapsulate the song into the main sections of INTRO, VERSE, CHORUS, BRIDGE, OUTRO

####I'll use classes for the sonic variety
Just as CSS rules are the adjectives for the differnt sonic patterns:
  - vocals
  - guitar
  - bass
  - drums

####Each Chord change will be an `<ol>` of 2 bars represented by `<li>` tags


