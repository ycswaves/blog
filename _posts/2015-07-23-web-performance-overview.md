---
layout: post
title: Web Performance Overview
---

#### The Performance Golden Rule
> Only 10-20% of the end user response time is spent downloading the HTML docu- ment. The other 80–90% is spent downloading all the components in the page.

Most of the time, it takes less effort to optimize on front-end to achieve a faster overall reponse time than on back-end. And before diving into the specific tasks to improve web performance. It's good to have a basic understanding of HTTP.

#### HTTP Overview
HyperText Transfer Protocol (HTTP) is how browsers and servers communicate with each other over the Internet and made up of requests and responses.

There are 7 types of requests:

- GET
- POST
- HEAD
- PUT
- DELETE
- OPTIONS
- TRACE

A request includes the type of request (e.g GET), a URL followed by headers. A response contains a status code, headers and a body.

The size of reponse can be reduced using compression if *both* the browser and server support it. For example, the browser announces its support of `gzip` using the `Accept-Encoding` header. The server identify compressed responses using the `Content-Encoding` header

Request:
{% highlight ruby %}
  Accept-Encoding: gzip
{% endhighlight %}

Response:
{% highlight ruby %}
  Content-Encoding: gzip
{% endhighlight %}
