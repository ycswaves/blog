---
layout: post
title: Make JavaScript and CSS External
---

In raw terms, placing JS and CSS files inline is faster because placing externally causes overhead of multiple HTTP requests. However, using external files in the real world generally produces faster page because of the benefit of being cached by the browser and the HTML document size will also be reduced by moving inline JS and CSS out of itself.

The key factor is the frequency with which external JS and CSS files are cached relative to the number of HTML documents requested. This can be gauged using following metrics:

- **Page Views:** fewer page views per user, the stronger the argument for inlining JS/CSS and vice versa.

- **Empty/Primed Cache:** if the nature of your site results in higher primed cache rates for your users, the benefit of using external files is greater. Users may show up with an empty cache, but make serveral subsequent page views with a primed cache.

- **Component Reuse:** if the JS and CSS files can be shared among multiple pages, using external files becomes more advantageous. However, the caveat here is that you may end up with either "few very big files" or "many small files". Only if you can find a balance that results in a high reuse rate, make JS/CSS external is the way to go, otherwise inlinging might make more sense.

### Home Pages
Home page might be the only exception where inlining is preferable. Take a look at the three metrics mentioned above from the perspective of home pages:

*Page Views:* Home pages have a high number of page views per month but often only one page view per session.  
*Empty/Primed Cache:* The primed cache percentage might be lower than other sites. For security rea- sons, many users elect to clear the cache every time they close the browser.  
*Component Reuse:* The reuse rate is low. Many home pages are the only page a user visits on the site, so there is really no reuse.

### The Best of Both Worlds
Even if all the factors point to inlining, it still feels inefficient to add all that JS and CSS to the page and not take advantage of the browser’s cache. Two techniques are described here that allow you to gain the benefits of inlining, as well as caching external files.

#### Post-Onload Download
Home page itself might be a good place to apply inlining, but also used to leverage external files for all secondary page views. This is accomplished by dynamically downloading the JS/CSS files in the home page (even if they might not be used in home page) after it has completely loaded. This places the external files in the browser's cache **in anticipation of** the user continuing on to other pages.

{% highlight html %}
<script type="text/javascript">
function postOnload() {
  setTimeout(downloadCompo, 1000);
  //one second delay to make sure the page is completely rendered
}

window.onload = postOnload;
function downloadCompo() {
  downloadJS("http://example.com/example.js");
  downloadCSS("http://example.com/example.css");
}

function downloadJS(url) {
  var script = document.createElement("script");
  script.src = url;
  document.body.appendChild(script);
}

function downloadCSS(url) {
  var css = document.createElement("link");
  css.rel = "stylesheet";
  css.type = "text/css";
  css.href = url;
  document.bodu.appendChild(css);
}
</script>
{% endhighlight %}

In case of double defination (same piece of code appears in both inlining and external files), inserting these components into an invisible `iframe` could tackle this problem.

#### Dynamic Inlining
