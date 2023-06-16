---
title: "深入浅出Webpack5模块联邦"
date: 2022-08-25T09:51:32+08:00
draft: false
hiddenFromHomePage: true
categories: ["note"]
tags: ["webpack"]
---
### 前言
很早之前就听说了webpack5的`模块联邦`，对它提供的应用之间模块级共享能力还挺感兴趣的，但一直还没有使用起来，本文会以5W1H的模式来围绕`模块联邦`做一个学习分享。

### Who
本文的目标读者是对`模块联邦`感兴趣但未使用过/不了解其原理的开发同学

### What
+ 是webpack5的新特性，允许多个webpack编译产物之间共享模块、依赖、甚至应用
+ 提供了一种轻量级的、在运行时，通过全局变量组合，在不同模块之前进行数据的获取
+ 提供了一种解决应用集的官方方案。每个构建都充当一个容器，也可将其他构建作为容器。通过这种方式，每个构建都能够通过从对应容器中加载模块来访问其他容器暴露出来的模块。

### Where
#### 每个页面单独构建
单页应用的每个页面都是在单独的构建中从容器暴露出来。主体应用程序也是独立构建，会将所有页面作为远程模块引用。通过这种方式，可以单独部署每个页面。在更新路由或者添加新路由时部署主体应用程序。主体应用程序将常用库定义为共享模块，以避免在页面构建中重复出现。

#### 将组件库作为容器
许多应用程序共享一个通用的组件库，可以将其构建成暴露所有组件的容器。每个应用程序使用来自组件库容器的组件。

#### When & Why
##### 公共模块复用
我们有A, B, C三个项目，针对这三个的一些公共模块，在进行组件/方法复用的时候，B项目抽离出公共的模块打包成npm包，通过发包的方式进行组件/方法复用。这样是没有问题的，但是当面临这个npm包的更新时，尤其是修复了bug时，我们不得不通知依赖模块进行升级，这样一个一个项目升级的模式肯定是低效率的。

**解决方案：**

#### 实现原理

#### How
[可参考的简单demo](https://juejin.cn/post/7189166861034979388)
[官方demo](https://stackblitz.com/github/webpack/webpack.js.org/tree/main/examples/module-federation?file=app2%2Fwebpack.config.js&terminal=start&terminal=)

#### 核心配置字段

+ name
name表示当前应用的别名，当作为remote时被host引用时需要在路径前加上这个前缀。

+ filename
filename表示remote应用提供给host应用使用时的入口文件，比如上面component应用设置的是remoteEntry，那么在最终的构建产物中就会出现一个`remoteEntry`的入口文件供main应用加载。

+ exposes
exposes表示remote应用有哪些属性、方法和组件需要暴露给host应用使用，他是一个对象，其中key表示在被host使用的时候的相对路径，value则是当前应用暴露出的属性的相对路径。

+ remote
remote 表示当前host应用需要消费的remote应用的以及他的地址，他是一个对象，key为对应remote应用的name值，这里要注意这个name不是remote应用中配置的name，而是自己为该remote应用自定义的值，value这是remote应用的资源地址。

+ shared
当前应用无论是作为host还是remote都可以共享依赖，而共享的这些依赖需要通过shared去指定

### 参考文档
+ [一文通透讲解webpack5 module federation](https://juejin.cn/post/7048125682861703181#heading-3)
+ [最详细的Module Federation的实现原理讲解](https://juejin.cn/post/7151281452716392462#comment)

