---
title: "为何使用vite"
date: 2021-08-13T09:51:32+08:00
draft: true
hiddenFromHomePage: true
categories: ["note"]
tags: ["vite"]
---
#### 为什么需要构建工具
在浏览器支持ES模块之前，js并没有提供原生机制让开发者以模块化的方式进行开发。这也正是我们对打包这个概念熟悉的原因：使用工具抓取、处理并将我们的源码模块串联成可以在浏览器中运行的文件。
#### 为什么是vite
##### 性能瓶颈
Vite使用生态系统中的新进展解决性能问题：
1. 浏览器开始原生支持ES模块
2. 越来越多js工具使用编译型语言编写

##### 服务器启动缓慢
当冷启动开发服务器时，基于打包器的方式启动必须优先抓取并构建你的整个应用，然后才能提供服务。

Vite通过在一开始将应用中的模块区分为**依赖**和**源码**两类，改进了开发服务器启动时间
+ **依赖** 大多为在开发时不会变动的纯js，一些较大的依赖处理代价也很高。依赖也通常会存在多种模块化格式，Vite使用[esbuild预构建模块](https://esbuild.github.io/)。esbuild使用Go编写，并且比以js编写的打包器预构建依赖快10-100倍。
+ **源码** 通常包含一些并非直接是js的文件，需要转换（例如JSX、CSS、或者是VUE组件等），时常会被编辑。Vite以[原生ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)方式提供源码。这实际上是让浏览器接管了打包程序的部分工作，Vite只需在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入代码，即只在当前屏幕上实际使用时才会被处理。

##### 更新缓慢
在Vite中，HMR是在原生ESM上执行的。当编辑一个文件时，Vite只需要精确地使已编辑的模块与其最近的HMR边界之间的链失活，使得无论应用大小如何HMR始终能保持快速更新。
同时利用HTTP头来加速整个也免得重新加载（缓存）