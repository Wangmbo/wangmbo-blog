---
title: "为何使用webpack"
date: 2021-08-25T09:51:32+08:00
draft: true
hiddenFromHomePage: true
categories: ["note"]
tags: ["webpack"]
----
使用`<script>`标签存在隐式依赖关系。

+ 无法直接体现，脚本的执行依赖于外部库
+ 如果依赖不存在，或者引入顺序错误，应用程序将无法正常运行
+ 如果依赖被引入，但是并没有使用，浏览器将被迫下载无用代码