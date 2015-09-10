---
layout: post
title: Minify JavaScript
category: notes
---

**Minification** is the practice of removing unnecessary characters (that includes all comments and unneeded whitespace characters) from code to reduce its size, thereby improving load times.

**Obfuscation** is an alternative optimization that can be applied to source code. Besides doing what minification does, it also converts function and variable names into smaller strings making the code more compact and harder to read. However, obfuscation has three main drawbacks:

- **Bugs**: There's a high chance of introducing errors into the code as a result of the obfuscation process itself.

- **Maintenance**: Since obfuscation changes JS symbols, any symbols that should not be changed (e.g API functions) must be tagged so that the obfuscator leaves them unaltered.

- **Debugging**: Obfuscated codes are more difficult to read, therefore, harder to debug in production environment.

Some popular tools for minifying JavaScript:

- [JSMin](http://crockford.com/javascript/jsmin)
- [Dojo (ShrinkSafe)](http://shrinksafe.dojotoolkit.org)

### The Savings
On average, JSMin reduced the size of JavaScript files by 21%, while Dojo Compressor achieved a 25% reduction. And generally one should choose minification over obfuscation, thus avoiding the possible problems that obfuscation can cause. The effectiveness on reduction of file size will be even closer for minification and obfuscation, when they are used together with gzip compression.

### Inline Scripts
Inline JavaScript blocks should also be minified, though it is less evident nowadays. In practice, minifying inlines is easier than that of externals because there is probably a version of JSMin in whatever page generation platform one uses. Once the functionality is available, all inlines can be minified before being echoed to the HTML document.

### Gzip and Minification
Gzip compression decreases file sizes more than minification. It could be questionable that minification is worthwhile when gzip compression has already been enabled.

| File size | JSMin  | Gzip | Savings |
| :-------: | :----: | :--: | :-----: |
| 85KB      |  No    |  No  |  -      |
| 68KB      |  Yes   |  No  |  21%    |
| 23KB      |  No    |  Yes |  73%    |
| 19KB      |  Yes   |  Yes |  78%    |

### Minifying CSS
The savings from minifying CSS are typically less than the savings from minifying JS because CSS generally has fewer comments and less whitespace. The greatest potential for size savings comes from optimizing CSS - merging identical classes, removing unused classes, etc.

