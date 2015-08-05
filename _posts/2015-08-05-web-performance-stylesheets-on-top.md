---
layout: post
title: Put Stylesheets at the Top
---

If certain portion of the web page, say a pop-up DIV, is only accessible when the parent page has been loaded, will putting the link to the stylesheet of that DIV at the bottom of the page help page load faster? **It may not help**

### Progressive Rendering
The problem with putting stylesheets near the bottom of the page is that it prohibits progressive rending in many browsers. Browsers block rendering to **avoid having to redraw elements of the page** if their style change. The browser delays showing any visible components while it and the user wait for the stylesheet at the bottom. 

