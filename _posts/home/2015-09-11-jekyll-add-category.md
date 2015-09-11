---
layout: post
title: Add Category in Jekyll
category: home
---

I've been using the Jekyll theme named ["Brume"](https://github.com/aigarsdz/brume) for quite a while. Besides doing some basic configurations in `_config.yml`, I rarely touch any other files due to the fact that I'm not familiar with the Jekyll internal mechanism (and Ruby), until yesterday I found that I probably need some categorization of my blog posts soon. Therefore, after some quick search on the Internet, I started tinkering with Jekyll's more "advanced" features for the first time.

The ultimate goal is to add a "Notes" tab in my blog's top navigation bar so that all the posts that are related to my readings go under that tab while the "Home" tab should only contains some other posts (e.g. the post your are reading now).

### Step 1: Adding "Notes" Tab
The first step is rather simple. Just locate `links.yml` under `_data` folder and add the *url* and *title* in between "Home" and "About" tab definitions so that the newly added "Notes" tab will become the 2nd tab in the navigation bar.
{% highlight html %}
- url: /blog/
  title: Home

- url: /blog/study-notes    -> add this line
  title: Notes

- url: /blog/about
  title: About
{% endhighlight %}

### Step 2: Restructure Post Folder
Previously all my posts were laid flat under `_posts` folder:
{% highlight html %}
|-- _posts
|   |-- 2015-07-23-web-performance-overview.md
|   |-- 2015-07-25-web-performance-fewer-request.md
|   |-- 2015-07-28-web-performance-cdn.md
|   |-- ...
{% endhighlight %}
What I need to do now is to add two new folders, `home` and `notes`, under `_posts` and put all the posts in the corresponding folders:
{% highlight html %}
|-- _posts
    |-- home
    |   |-- 2015-09-11-jekyll-add-category.md
    |-- notes
        |-- 2015-07-23-web-performance-overview.md
        |-- 2015-07-25-web-performance-fewer-request.md
        |-- 2015-07-28-web-performance-cdn.md
{% endhighlight %}
Putting the posts under different folders doesn't mean the categorization is done. Jekyll is not that smart yet to differentiate each category just by the folder structure - we need a bit of coding.

### Step 3: Filter Posts by Category
In Step 1, we have specified this path: `/blog/study-notes`. So we need create the folder `study-notes` under blog's root folder and copy the `index.html`, the template that the page under "Home" tab is using, into the `study-notes` folder. Basically this template will render a catalog page listing the title and date of all the posts.

Open `study-notes/index.html` and locate this line:
{% highlight html %}
{% raw %}
{% for post in site.posts %}
{% endraw %}
{% endhighlight %}
This line tells Jekyll to find all posts in the current blog. So to achieve what we want, we need to replace this line with following:
{% highlight html %}
{% raw %}
{% for post in site.categories.notes %}
{% endraw %}
{% endhighlight %}
This is to tell Jekyll that we only want the posts belong to the category of "notes" to be listed in the catalog. The syntax is `site.categories.CATEGORY_NAME`. Therefore in `index.html` under root folder, the file that defines the "Home" page's template, this line will become:
{% highlight html %}
{% raw %}
{% for post in site.categories.home %}    -> replace "notes" with "home"
{% endraw %}
{% endhighlight %}

### Step 4: Give Each Post the Category Name
So the last step is just to add a "category" attribute in the post header section. For example, in my case, the post "Web Performance Overview" should belong to "notes":
{% highlight html %}
---
layout: post
title: Web Performance Overview
category: notes    -> category attribute goes here
---
{% endhighlight %}
So now when we click the "Notes" tab, only the post "Web Performance Overview" will be listed in the catalog. Just assign the "category" attribute to the rest of the post, we will get what we want. It's that simple!
