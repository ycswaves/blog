---
layout: post
title: Gzip Components
category: notes
---

### How Compression Works
Starting with HTTP/1.1, web clients indicate support for compression with the `Accept-Encoding` header in the HTTP request.

```html
Accept-Encoding: gzip, deflate
```

Web server sees this header will compress the reponse using one of the methods listed and nofify the client via the `Content-Encoding` header in the reponse

```html
Content-Encoding: gzip
```

> Browsers that support `deflate` also support `gzip` but not the other way around, so `gzip` is the preferred method of compression.

### What to Compress
- HTML documents
- Scripts
- Stylesheets
- XML/JSON reponses  

#### Not to compress:  
- Image
- PDF

As they are already compressed. Trying to gzip wastes CPU resources and could also potentially increase file sizes.

#### Cost of gzipping
It takes additional CPU cycles on both server-side and client-side to compress and decompress the gzipped files. Generally, it's worth gzipping any file greater than **1 or 2K**.
