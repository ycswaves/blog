---
layout: post
title: Put Stylesheets at the Top
category: notes
---

If certain portion of the web page, say a pop-up DIV, is only accessible when the parent page has been loaded, will putting the link to the stylesheet of that DIV at the bottom of the page help page load faster? **It may not help**

### Progressive Rendering
The problem with putting stylesheets near the bottom of the page is that it prohibits progressive rending in many browsers. Browsers block rendering to **avoid having to redraw elements of the page** if their style change. The browser delays showing any visible components while it and the user wait for the stylesheet at the bottom. 

### CSS at the Top
To avoid the blank white screen at the very first beginning of page loading, especially in IE, move the stylesheet to the top in the HTML's `<head>` section so that the page can render progressively.

#### Two ways of including a stylesheet in HTML
Link tag:
```html
<link rel="stylesheet" href="styles1.css">
```

@import:
```html
<style>
  @import url("styles2.css");
</style>
```

Benefits of `<link>` over `@import`:

- `@import` must precede all other rules, otherwise it might be ignored
- `@import` may still be downloaded much later even if it's in `<head>`

Cases when use `@import` is appropriate:

- Use inside `@media` to import device specific stylsheets
- Import Google fonts to avoid paste a `link` into every page using that stylesheet
- There are building tools that will concat all stylesheets and replace `@import`
