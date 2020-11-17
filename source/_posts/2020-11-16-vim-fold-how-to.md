---
title: Vim Fold How-To
date: 2020-11-16 21:11:49
tags: [vim]
---

There are multiple folding methods in Vim, and the two most frequently used methods I've been using are:

# Manual
Folds must be defined by entering commands such as `zf`, which is the default folding method if there's no `foldmethod` specified.
You can use vim command:
```
set foldmethod=[manual|indent|syntax|...]
```
to specify the folding method in the current window.

In normal mode, a fold can be created by typing:
```
zf{motion}
```
Example 1 - Folds the current line along with the following 3 lines
{% asset_img zf3j.gif folds the current line along with the following 3 lines %}

Example 2 - Folds the current line till the end of the file
{% asset_img zfG.gif folds the current line till the end of the file %}

# Syntax
Syntax fold can be activated by running `setlocal foldmethod=syntax` in Vim command mode. In this mode, fold can be created based on the syntax. For example, you can create a fold around a code block or function declaration.

Example 3 - Fold a block
{% asset_img zcBlock.gif create a fold around a code block %}

Example 4 - Fold a variable assignment
{% asset_img zcAssignment.gif create a fold around a variable assignment %}

# Common commands
- `zr`: reduces fold level throughout the buffer
- `zR`: opens all folds
- `zm(inimize)`: increases fold level throughout the buffer
- `zM`: folds everything all the way
- `za(lternate)`: alternate a fold your cursor is on
- `zA`: alternate a fold your cursor is on recursively
- `zc(lose)`: close a fold your cursor is on
- `zC`: close a fold your cursor is on recursively
- `zo(pen)`: open one fold under the cursor.
- `zO`: open all fold under the cursor.

