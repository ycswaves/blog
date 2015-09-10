---
layout: post
title: Avoid Redirects
category: notes
---

A `redirect` is used to reroute users from ono URL to another, `301` and `302` are the most popular ones. Redirects are usually done for HTML documents, but may also be used when requesting components in the page (images, scripts, etc.).

### Types of Redirects
When web servers return a redirect to the browser, the response has a status code in the 3xx Ramge:

- 300 Multiple Choices (based on Content-Type)
- 301 Moved Permanently
- 302 Moved Temporarily (a.k.a Found)
- 303 See Other (clarification of 302)
- 304 Not Modified (not really a redirect, used in response to condtional GET)
- 305 Use Proxy
- 306 (no longer used)
- 307 Temporary Redirect (clarification of 302)