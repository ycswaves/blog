---
layout: post
title: Avoid CSS Expressions
---

CSS expressions are a powerful but dangerous way to set CSS properties dynamically. They are supported in IE version 5 and later ([but are no longer supported in IE8 and later](https://msdn.microsoft.com/en-us/library/ms537634(v=vs.85).aspx)).

{% highlight html %}
background-color: expression( (new Date()).getHours()%2 ? "#B8D4FF" : "#F08A00");
{% endhighlight %}

The above example shows how background color could be set to alternate every hour using CSS expression. The expression method is ignored by all browsers other than IE, so it's a useful tool for setting properties in IE to create consistent experience across browsers. For instance, IE6 doesn't support `min-width`, you may use following css rules to achieve that: 

{% highlight html %}
width: expression( document.body.clientWidth < 600 ? "600px" : "auto" );
min-width: 600px;
{% endhighlight %}

Browsers support `min-width` will ignore the first rule while IE6 ignore the `min-width` property and sets the `width` property dynamically based on the width of the document.

### Updating Expressions
The problem with expressions is that they're evaluated **too frequently**. They're evaluated when:

- Page is rendered and resized
- Page is scrolled
- User moves the mouse over the page

Moving the mouse around the page can easily generate >10,000 evaluations. Worst of all, clicking in the text input field locks up IE and you have to kill the process.

### Working Around
The working around is to use JavaScript to achieve the dynamic change, either define a function to do a one-time expressions or an event handler to respond to the intended action only to avoid evaluation of the expression during unrelated events.