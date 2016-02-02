---
layout: post
title: A Better Hex Coordinate System
date: '2011-07-31T16:16:00.000-07:00'
author: Morgan Freshour
tags: 
modified_time: '2011-07-31T16:16:51.833-07:00'
blogger_id: tag:blogger.com,1999:blog-3127719415764963943.post-7308312372775613468
blogger_orig_url: http://hillbilly-nerd.blogspot.com/2011/07/better-hex-coordinate-system.html
---

When I started working on <a href="https://github.com/mgfreshour/hexwar">Hexwar</a>, I used the simple approach to displaying hexagons from the article <a href="http://www-cs-students.stanford.edu/~amitp/Articles/GridToHex.html">Pixel Coordinates to Hexagonal Coordinates</a>.  The problem with this coordinate system is that it's difficult to perform functions for distance on it.  I ended up walking the entire grid from the starting hex and making an array of distances.  This proved faulty (due to my faulty walking algorithm) and I decided to move to the coordinate system similar to <a href="http://www-cs-students.stanford.edu/~amitp/Articles/HexLOS.html">Clark Verbrugge’s Hex Grids</a>.  The major difference being that I want my hexes to point horizontally instead of vertically.  (I considered using the <a href="http://stackoverflow.com/questions/2459402/hexagonal-grid-coordinates-to-pixel-coordinates">three coordinate hex space model</a>, but that'd require a change to a lot of my code.)


## Explanation of Coordinate Systems
Understanding this, one must see there are three different coordinate systems at work here.  



### Hex Space
First comes the hex space.  This is the game world's coordinate system.  It is the one which all the game mechanics rely upon.  It looks like :

<pre>Hex Space
                      __   5
                   __/D \__   4
                __/  \__/  \__   3      "Y" coord
             __/  \__/  \__/  \__   2
          __/A \__/  \__/  \__/  \__   1
       __/  \__/  \__/E \__/B \__/  \__   0
      /  \__/G \__/  \__/  \__/F \__/C \
      \__/  \__/  \__/  \__/  \__/  \__/
         \__/  \__/  \__/  \__/  \__/   5
            \__/  \__/  \__/  \__/   4
               \__/  \__/  \__/    3
                  \__/  \__/    2     "X" coord
                     \__/    1
                         0  
Such that A =(2,5); B =(4,2); C =(5,0); D =(5,5); E =(3,3); F =(4,1); G =(1,4)  
</pre>

### Array Space
Next is the array space.  This is a translation of the hex space into an orthogonal space.  I've made it the same as the original hex space I was using.  It begins with (0,0) in the top left and extends positively to the right and down.  It looks like :

<pre>Array Space

 __    __    __    __
/M \__/O \__/  \__/  \__
\__/N \__/  \__/  \__/  \
/  \__/  \__/  \__/  \__/
\__/  \__/  \__/  \__/  \
/P \__/R \__/  \__/  \__/
\__/Q \__/  \__/  \__/  \
/  \__/  \__/  \__/  \__/
\__/  \__/  \__/  \__/

Such that M =(0,0); N =(1,0); O =(2,0); P =(0,3); Q =(1,3); R =(2,3)
</pre>The reason for this space is that computer screens are square, not diamond shaped.  The hex space as described above would be displayed such that a large percentage of the screen would be unused, or a large percentage of world would be unseen.



### Screen Space
Finally is the screen space.  This represents the pixels on the screen.  This could also be a viewport space if I need to implement scrolling, but I currently want the entire map to appear on screen at once. This starts at (0,0) for the pixel in the top left and proceeds positively to the right and down.



## Converting Between Coordinate Systems

### Between Array and Hex Coordinates
{% highlight javascript %}
Hexwar.Hex.convertHexToArrayCoords = function(x, y) {
 return {
    x: x + y
  , y: Math.floor((x+y)/2) -y
 };
}

Hexwar.Hex.convertArrayToHexCoords = function(x, y) {
 return {
    x: y + Math.ceil(x/2)
  , y:-1*(y - Math.floor(x/2))
 };
}
{% endhighlight %}



### Between Screen and Array Coords
{% highlight javascript %}
Hexwar.Hex.convertArrayToScreenCoords = function(x,y) {
 return {
  x: (x * Hexwar.Hex.HEX_SIDE * 1.5),
   y: (y * Hexwar.Hex.HEX_HEIGHT + (x % 2) * Hexwar.Hex.HEX_HEIGHT / 2)
 };
}

Hexwar.Hex.convertScreenToArrayCoords = function(screenX, screenY) {
 // ----------------------------------------------------------------------
  // --- Determine coordinate of map div as we want the click coordinate as
 // --- we want the mouse click relative to this div.
 // ----------------------------------------------------------------------
 var posx = screenX;
 var posy = screenY;

 // ----------------------------------------------------------------------
 // --- Convert mouse click to hex grid coordinate
 // --- Code is from http://www-cs-students.stanford.edu/~amitp/Articles/GridToHex.html
 // ----------------------------------------------------------------------
 x = (posx - (Hexwar.Hex.HEX_HEIGHT/2)) / (Hexwar.Hex.HEX_HEIGHT * 0.75);
 y = (posy - (Hexwar.Hex.HEX_HEIGHT/2)) / Hexwar.Hex.HEX_HEIGHT;
 z = -0.5 * x - y;
 y = -0.5 * x + y;

 ix = Math.floor(x+0.5);
 iy = Math.floor(y+0.5);
 iz = Math.floor(z+0.5);
 s = ix + iy + iz;
 if (s) {
  abs_dx = Math.abs(ix-x);
  abs_dy = Math.abs(iy-y);
  abs_dz = Math.abs(iz-z);
  if (abs_dx >= abs_dy && abs_dx >= abs_dz) {
   ix -= s;
  } else if (abs_dy >= abs_dx && abs_dy >= abs_dz) {
   iy -= s;
  } else {
   iz -= s;
  }
 }

 // Done!
 return {
  x: ix,
  y: (iy - iz + (1 - ix % 2 ) ) / 2 - 0.5
 }

}
{% endhighlight %}


## Calculating the distance between two hexes
Finally for the thing that I did the switch for.  This hex coordinate system allows a quick (compared to walking the whole map) calculation of distance between two hexes.  It also allows for some Line-of-Site calculations, but I don't currently need those.  Anyways, the code :

{% highlight javascript %}
Hexwar.Hex.calculateDistanceHexSpace = function(a, b) {
 var dx = b.x - a.x;
 var dy = b.y - a.y;

  if (Math.sign(dx) != Math.sign(dy)) {
  return Math.max(Math.abs(dx), Math.abs(dy));
 } else {
  return Math.abs(dx) + Math.abs(dy);
 }
}
{% endhighlight %}


Oh well, that's it for now!
