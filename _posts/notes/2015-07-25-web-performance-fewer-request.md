---
layout: post
title: Make Fewer HTTP Request
category: notes
---

There are several techniques that help reduce HTTP requests, including:

- Using image maps (not very common nowadays)
- CSS sprites (quite common)
- inline images
- combined scripts and stylesheets

### 1. Image Map
There are 2 types of them:  
**Server-side** image maps: by submitting all clicks to the same destination URL, passing along the x,y coordinates of where the user clicked.  
**Client-side**: The mapping is achieved via HTML's `<map>` tag.  
Example:
{% highlight html %}
<img usemap="#map1" border=0 src="/images/imagemap.gif">
<map name="map1">
  <area shape="rect" coords="0,0,31,31" href="home.html" title="Home">
  <area shape="rect" coords="36,0,66,31" href="gifts.html" title="Gifts">
  <area shape="rect" coords="71,0,101,31" href="cart.html" title="Cart">
  <area shape="rect" coords="106,0,136,31" href="settings.html" title="Settings">
  <area shape="rect" coords="141,0,171,31" href="help.html" title="Help">
</map>
{% endhighlight %}

### 2. CSS Sprite
The concept of CSS Sprite is to put all individual images into one big image and select part of it where you want it to be displayed as the background image of a html element (that supports background images like a `<span>` or `<div>`).

To "select" which image to show from the big image file, use the CSS `background-position` property
{% highlight html %}
<div style="background-image: url('css_sprites.gif');
            background-position: -260px -90px;
            width: 26px; height: 24px;">
</div>
{% endhighlight %}
The 2 values, "-260px" and "-90px", defines the top-left position of the sprite-image. In this case, to display the target small image, it requires the sprite-image to move left by 260px and up by 90px. And that's why you never see positive values for `background-position`.

### 3. Combined Scripts and Stylesheets
Generally, using external scripts and stylesheets is better for performance. From software engineering's point of view, it's good to modularize the code by breaking it into many small files, which means more HTTP requests.

Combining files is easy, and minifying the files can also be done along with the combining process. However it will be difficult if different pages require different combinations of modules. (Maybe take a look at RequireJS)



