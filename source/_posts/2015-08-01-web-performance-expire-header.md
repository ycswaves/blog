---
layout: post
title: Add an Expires Header
category: notes
---

### Expires Header

By using a future `Expires` header, it makes images, scripts or stylesheets cacheable and avoids unnecessary HTTP requests on subsequent page views.

**Browsers** use a cache to reduce the number of HTTP requests and decrease the size of HTTP response, thus making web pages load faster.  
**Web servers** use the `Expires` header to tell the browser that it can use the current copy of a resource file until the specified time.

```html
Expires: Sun, 1 Aug 2015 14:30:00
```

### Max-Age and mod_expires

The `Cache-Control` header was introduced, as an alternative to the `Expires` header, in HTTP/1.1, to overcome limitations with the `Expires` header.

```html
Cache-Control: max-age=315360000
```

| Expires                                | Cache-Control                                        |
|----------------------------------------|------------------------------------------------------|
| Specific date                          | Use `max-age` to define fresheness window in seconds
| Expire date constantly checked         |
| Once expire, new date must be provided

If both are present, `max-age` directive will override the `Expires` header. 

### Get user updated when cached files change
To ensure users get the latest version of a file, change the filename in all HTML pages:
> The most effective solution is to change any links to them; that way, completely new representations will be loaded fresh from the origin server. 

Tips: Embed the version number in the filename (e.g. *my_js_file_v2.0.1.js).
