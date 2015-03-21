title: "Shebang"
date: 2015-03-01 11:25:35
tags: linux
categories: 操作系统基础
---

[wiki](http://zh.wikipedia.org/wiki/Shebang)链接

在计算机科学中，Shebang（也称为Hashbang）是一个由井号和叹号构成的字符序列（**#!**），其出现在文本文件的第一行的前两个字符。 在文件中存在Shebang的情况下，类Unix操作系统的程序载入器会分析Shebang后的内容，将这些内容作为解释器指令，并调用该指令，并将载有Shebang的文件路径作为该解释器的参数。

```
#!/bin/sh
```

开头的文件在执行时会实际调用/bin/sh程序（通常是Bourne shell或兼容的shell，例如bash、dash等）来执行。

## 语法

Shebang这一语法特性由`#!`开头，即井号和叹号。 在开头字符之后，可以有一个或数个空白字符，后接解释器的绝对路径，用于调用解释器。Shebang行也可以包含需要传递到解释器的特定选项。然而，选项传递的方式随实现的不同而不同。

使用`#!/usr/bin/env`脚本解释器名称是一种常见的在不同平台上都能正确找到解释器的办法。

**Node global bin**：`#!/usr/bin/env node`
