---
layout: post
title: Make JavaScript and CSS External
---

In raw terms, placing JS and CSS files inline is faster because placing externally causes overhead of multiple HTTP request. However, using external files in the real world generally produces faster page because of the benefit of being cached by the browser and the HTML document size will also be reduced by moving inline JS and CSS out of itself.

The key factor is the frequency with which external JS and CSS files are cached relative to the number of HTML documents requested. This can be gauged using following metrics:

- **Page Views:** fewer page views per user, the stronger the argument for inlining JS/CSS and vice versa.
- **Empty/Primed Cache:** if the nature of your site results in higher primed cache rates for your users, the benefit of using external files is greater. Users may show up with an empty cache, but make serveral subsequent page views with a primed cache.
- **Component Reuse:** if the JS and CSS files can be shared among multiple pages, using external files becomes more advantageous. However, the caveat here is that you may end up with either "few very big files" or "many small files". Only if you can find a balance that results in a high reuse rate, make JS/CSS external is the way to go, otherwise inlinging might make more sense 