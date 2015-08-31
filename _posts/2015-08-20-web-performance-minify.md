---
layout: post
title: Minify JavaScript
---

**Minification** is the practice of removing unnecessary characters (that includes all comments and unneeded whitespace characters) from code to reduce its size, thereby improving load times.

**Obfuscation** is an alternative optimization that can be applied to source code. Besides doing what minification does, it also converts function and variable names into smaller strings making the code more compact and harder to read. However, obfuscation has three main drawbacks:

- **Bugs**: There's a high chance of introducing errors into the code as a result of the obfuscation process itself.

- **Maintenance**: Since obfuscation changes JS symbols, any symbols that should not be changed (e.g API functions) must be tagged so that the obfuscator leaves them unaltered.

- **Debugging**: Obfuscated codes are more difficult to read, therefore, harder to debug in production environment.
