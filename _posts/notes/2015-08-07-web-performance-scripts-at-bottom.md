---
layout: post
title: Put Scripts at the Bottom
category: notes
---

With scripts, progressive rendering is blocked for all content *below* the script. Moving scripts lower in the page means more content is rendered progressively.

### Parallel Downloads

Response time mainly affected by the number of compoments (resource files) in the page. Each component generate an HTTP request. Browser performs HTTP requests in parallel but **can't** download them all at once.

HTTP/1.1 suggests browsers to download two componnents in parallel per hostname. If a web page evenly distribute its components across two hostnames, the overall response would be about twice as fast. 

Limiting parallel downloads to two per hostname is a guideline. However, users can modify this setting in their browsers, and front-end engineers could use CNAMEs(DNS aliases) to split their components across multiple hostnames. Too many parallel downloads can degrade performance. Research at Yahoo! shows that splitting components across **two** hostnames leads to better performance than using 1, 4 or 10 hostnames.

### Scripts Block Downloads
The benefits of parallel downloading is clear. However, parallel downloading is **disabled** while a **script** is downloading, even on different hostnames because:

- The script may use `document.write` to alter the page content, so the browser waits to make sure the page is laid out appropriately.
- To guarantee that the scripts are executed in the proper order, otherwise executing them out of order would result in JavaScript errors.

### Putting It in Perspective
In some situations, it's not easy to move scripts to the bottom. For example if the script use `document.write` to insert part of the page's content. There might also be scoping issues. If that script doesn't contain `document.write`, you may use *deferred* scripts as a clue to browers that they can continue rendering. However, in Firefox, even deferred scripts block rendering and parallel downloads.