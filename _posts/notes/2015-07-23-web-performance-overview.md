---
layout: post
title: Web Performance Overview
category: notes
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

#### Compression
The size of reponse can be reduced using compression if **both** the browser and server support it. For example, the browser announces its support of `gzip` using the `Accept-Encoding` header. The server identify compressed responses using the `Content-Encoding` header.

Request:
{% highlight ruby %}
  Accept-Encoding: gzip
{% endhighlight %}

Response:
{% highlight ruby %}
  Content-Encoding: gzip
{% endhighlight %}

#### Conditional GET
The browser may cache certain resource files like `.css` and `.js` files. To make sure those cached files are not modified since the time of last retrieval, a conditional GET request is sent by using `If-Modified-Since` header.

{% highlight ruby %}
  If-Modified-Since: Fri, 24 Jul 2015 04:15:54 GMT
{% endhighlight %}

And if the requested resource is not expired, server will respond with a '304 Not Modified' status and **skip sending the reponse body**, resulting in a smaller and faster response.

#### Keep-Alive
HTTP is built on top of TCP. In early days, every HTTP request required opening a new socket connection. Thus *Persistent Connections* (a.k.a "Keep-Alive" in HTTP/1.0) was introduced to let browsers make multiple requests over a single connection.

{% highlight ruby %}
  Connection: keep-alive
{% endhighlight %}

Technically, `keep-alive` is not required in HTTP/1.1, but most browsers and servers still include it.



