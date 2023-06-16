---
title: "webpack基本概念"
date: 2021-08-25T09:51:32+08:00
draft: true
hiddenFromHomePage: true
categories: ["note"]
tags: ["webpack"]
---
### 首先我们了解一下模块的概念
开发者将程序分解为功能离散的chunk，我们称之为模块
Node.js从一开始就支持模块化编程，但是web的模块化还在缓慢的支持中，webpack将模块的概念应用到项目的任何文件中

### 何为webpack模块
与node.js模块相比，webpack模块能以各种方式表达它们的依赖关系
+ es6 module `import` 语句
+ CommonJS `require`语句
+ AMD
+ css/sass/less 文件中的`@import`语句
+ stylesheet `url()` 或者 HTML `<img src=...>`文件中的图片链接

通过`loader`可以使webpack支持多种语言和预处理器语法编写的模块。loader向webpack描述了如何处理非原生模块，并将相关依赖引入到你的bundles中。webpack社区已经为各种流行的语言和预处理器创建了loader。
webpack提供了可定制的能力，通过自定义loader实现

### 依赖图
每当一个文件依赖另一个文件时，webpack都会将文件视为直接存在依赖关系。这使得webpack可以获取非代码资源，如images或者web字体等。并可将它们作为依赖提供给应用程序。
当webpack处理应用程序时，它会根据命令行参数中或配置文件中定义的模块列表开始处理。从入口开始，webpack会递归的构建一个依赖关系图，这个依赖图包含着应用程序中所需的每个模块，然后将所有模块打包为少量的bundle，可由浏览器加载。

### 模块
ES2015中的import和export语句已经被标准化，虽然大多数浏览器还无法支持它们，但是webpack却能够提供开箱即用般的支持。事实上，webpack在幕后将会转译，以便浏览器可以执行。webpack不会更改代码中除import和export语句以外的部分。如果我们在使用其他es2015特性，确保在webpack loader系统中使用了一个像是Babel的transpiler(转译器)。

### 使用配置文件
webpack 默认使用 `webpack.config.js` 作为配置文件，我们可以使用--config选项传递任何名称的配置文件。

### 管理资源
在webpack出现之前，前端开发人员会使用grunt和gulp等工具来处理资源，并将它们从`/src`文件夹移动到`/dist`目录中。js模块也遵循同样的方式，但是像webpack这样的工具，将动态打包所有的依赖创建所谓的依赖图。这样每个模块都可以明确表述它自身的依赖，可以避免打包未使用的模块。
+ 模块loader可以链式调用。链中每个loader都将对资源进行转换。链会逆序执行，第一个loader将其结果传递给下一个loader，依次类推。最后，webpack期望链中的最后loader返回js。
+ https://baijiahao.baidu.com/s?id=1740674963256534910&wfr=spider&for=pc css相关loader常用配置
+ 像background和icon这样的图像，在webpack5中，可以使用内置的Asset Modules，我们可以轻松的将这些内容混入我们的系统中。loader会识别这是一个本地文件，并将路劲替换为output目录中图像的最终路径。
+ https://www.webpackjs.com/guides/asset-modules/ 资源模块是一种模块类型，他允许使用资源文件（字体，图标等）而无需配置额外的loader。

### 开发环境
+ mode 设置为 `development`
+ 使用sourcemap
+ watch mode 如果文件被更新，代码将被重新编译，为了看到修改后的实际效果，还需要刷新浏览器
+ `webpack-dev-server` 提供了一个基本的web server 并且具有实时重新加载功能


### 代码分离
代码分离是webpack中最引人注目的特性之一。此特性能够把代码分离到不同的bundle中，然后可以按需加载或并行加载这些文件。代码分离可以用于获取更小的bundle，以及控制资源加载优先级，如果使用合理，会极大影响加载时间
+ 入口起点：使用`entry`配置手动地分离代码
  + 重复模块会被引入到各个bundle中
+ 防止重复：使用Entry dependencies 通过配置`dependOn` 或者 `SplitChunksPlugin`去重和分离chunk
+ 动态导入：通过模块的内联函数调用来分离代码 TODO how to do

### 模式
提供`mode`配置，告知webpack使用相应模式的内置优化
+ development
+ production

### 环境变量
+ webpack 命令行环境配置的`--env`参数，可以传入任意数量的环境变量

### 依赖管理
可以通过`require.context()`函数来创建自己的context，支持三个参数，搜索的目录，是否搜索子目录，匹配文件的正则表达式

### 模块热替换
模块热替换功能会在应用程序运行过程中、替换、添加或删除模块而无需重新加载整个页面：
+ 保留在完全重新加载页面期间丢失的应用程序状态
+ 只更新变更内容，以节省宝贵的开发时间
+ 在源代码中CSS/JS产生修改时，会立刻在浏览器中进行更新，这几乎相当于在浏览器devtools直接更改样式

#### 在应用程序中
1. 应用程序要求 HMR runtime 检查更新
2. HMR runtime 异步地下载更新，然后通知应用程序
3. 应用程序要求HMR tuntime应用更新
4. HMR runtime 同步地应用更新
你可以设置HMR，以使此进程自动触发更新，或者你可以选择要求在用户交互时进行更新

#### 在compiler中
除了普通资源，compiler需要发出'update'，将之前的版本更新到新的版本。'update'由两部分组成：
1. 更新后的manifest(JSON)
2. 一个或多个updated chunk(JavaScript)

manifest包括新的compilation hash 和所有的 updated chunk 列表。每个chunk都包含着更新模块的最新代码（或者一个flg用于表明此模块是否需要被移除）
compiler会确保在这些构建之间的模块ID和chunk ID保持一致。通常将这些ID存储在内存中，（例如，使用`webpack-dev-server`），但是也可能会将它们存储在一个JSON文件中

#### 在模块中
HMR是可选功能，只会影响包含HMR代码的模块。举个例子，通过`style-loader`为style追加补丁。为了运行追加补丁，`style-loader`实现了HMR接口，当它通过HMR接收到更新，它会使用新的样式替换旧的样式。
类似的，当在一个模块中实现了HMR接口，你可以描述出当模块被更新后发生了什么。然后再多数情况下，不需要在每个模块中强行写入HMR代码。如果一个模块没有HMR处理函数，就会冒泡。
这意味着某个单独处理函数能够更新整个模块树。如果在模块树的一个单独模块被更新，那么整组依赖模块都会被重新加载

#### 在runtime中
对于模块系统运行时，会发出额外代码，来跟踪模块`parent`和`chidlren`关系。在管理方面，runtime支持两个办法`check`和`apply`。

`check`方法，发送一个HTTP请求来更新manifest。如果请求失败，说明没有可用更新。如果请求成功，会将updated chunk 列表与当前的loaded chunk 列表进行比较。每个loaded chunk 都会下载相应的updated chunk。当所有更新chunk完成下载，runtime就会切换到`ready`状态。

`apply` 方法，将所有updated module 标记为无效。对于每个无效module, 都需要在模块中有一update handler 或者在此模块父级模块有update handler 否则，会进行无效标记冒泡，并且父级也会被标记为无效。继续每个冒泡，直到到达应用程序入口七点。
或者到达带有update handler 的module。之后，所有无效的module都会被 dispose handler 处理和接触加载。
然后更新当前hash，并调用所有`accept`handler。runtime切换回idle状态，一且照常继续。

### Tree Shaking
通常用于移除JavaScript上下文的未引用代码(dead-code)，它依赖于ES2015模块语法的静态结构特性，例如`import`和`export`。这个术语和概念实际上是由ES2015模块打包工具rollup普及起来的。
`静态结构`：指的是模块中的导入导出只能使用字面量，不能使用变量或表达式。这意味着模块的依赖关系可以在静态分析时确定，因此可以更容易地优化和删除未使用的代码。这是实现tree shaking的基础。

### 生产环境
development和production这两个环境下的构建目标存在着巨大差异。在开发环境中，我们需要：强大的source map和一个有着live reloading 或 hot module replacement(热模块替换)能力的localhost server。
而生产环境目标则转移至其他方面，关注点在于压缩bundle、更轻量的source map、资源优化等，通过这些优化方式改善加载时间。由于要遵循逻辑分离，通常建议为每个环境编写彼此独立的webpack配置。

虽然，以上我们将生产环境和开发环境做了细微区分，但是，请注意，我们还是会遵循不重复原则。保留一个common配置，为了将这些配置合并在一起，我们将使用一个名为webpack-merge工具。

### ESModule
ECMAScript模块是在Web中使用模块的规范。所有现代浏览器均支持此功能，同时也是在Web中编写模块化代码的推荐方式。
Node.js通过设置`pacakge.json`中的属性来显式设置文件模块类型。在package.json中设置`"type":"module"`会强制package.json下的所有文件使用ESMdule模块。设置`"type":"commonjs"`将会强制使用CommonJS模块。
除此之外，还可以通过使用`.mjs`或`.cjs`扩展名来设置模块类型。

### ProvidePlugin 预置全局变量

### Module Federation
多个独立部署的构建可以组成一个应用程序，这些独立的构建之间不应该存在依赖关系，因此可以单独开发和部署他们。
> webpack5模块联邦让webpack达到了线上Runtime的效果，让代码直接在项目间利用CDN直接共享，不再需要本地安装NPM包、构建再发布了

#### 底层概念
我们区分本地模块和远程模块。本地模块即为普通模块，是当前构建的一部分。远程模块不属于当前构建，并在运行时从所谓的容器加载。
加载远程模块被认为是异步操作。当使用远程模块时，这些异步操作将被放置在远程模块和入口之间的下一个chunk的加载操作之中。如果没有chunk加载操作，就不能使用远程模块。

chunk的加载操作通常是通过调用`import()`实现的，但也支持像`require.ensure`或`require([...])`之类的旧语法。
容器是由容器入口创建的，该入口暴露了对特定模块的异步访问。暴露的访问分为两个步骤：
1. 加载模块（异步的）
2. 执行模块（同步的）

步骤1将在chunk加载期间完成。步骤2将在与其他（本地与远程）的模块交错执行期间完成。这样一来执行顺序不受模块从本地转换为远程或从远程转为本地的影响。
容器可以嵌套使用，容器可以使用来自其他容器的模块。容器之间也可以循环依赖。

#### 高级概念
每个构建都充当一个容器，也可将其他构建作为容器。每个构建都能够从对应容器中加载模块来访问其他容器暴露出来的模块。

共享模块是指既可重写的又可作为向嵌套容器提供重写的模块。它们通常指向每个构建中相同的模块，例如相同的库。

packageName选项允许通过设置包名来查找所需的版本。默认情况下，它会自动推断模块请求，当想禁用自动推断时，将requiredVersion设置为`false`

#### 构建块
+ ContainerPlugin
使用指定的公开模块来创建一个额外的容器入口
+ ContainerReferencePlugin
将特定的引用添加到作为外部资源的容器中，并允许从这些容器中导入远程模块。它还会调用这些容器`override`API来为它们提供重载。
+ ModuleFederationPlugin
组合以上两个插件

#### 概念目标
+ 既可以暴露，又可以使用webpack支持的任何模块类型
+ 代码加载应该并行加载所需的所有内容
+ 从使用者到容器的控制
  + 重写模块是一种单向操作
  + 同级容器不能重写彼此的模块
+ 概念适用于独立环境
  + 可用于web,Node.js等
+ 共享中的相对和绝对请求
  + 会将相对路径解析到config.context
+ 共享中的模块请求
  + 只在使用时提供
  + 会匹配构建中所有使用的相等模块请求
  + 将提供所有匹配模块
+ 共享中尾部带有`/`的模块请求将匹配所有具有这个前缀的模块请求

#### 用例
##### 每个页面单独构建
单页应用的每个页面都是在单独的构建中从容器暴露出来。主体应用程序也是独立构建，会将所有页面作为远程模块引用。通过这种方式，可以单独部署每个页面。在更新路由或添加新路由时部署主体应用程序。
主体应用程序将常用库定义为共享模块，以避免在页面构建中出现重复。

##### 将组件库作为容器
许多应用程序共享一个通用的组件库，可以将其构建成暴露所有组件的容器。每个应用程序使用来自组件库容器的组件。可以单独部署对组件库的更改，而不需要重新部署所有应用程序。应用程序自动使用组件库的最新版本。

##### 动态远程容器
该容器接口支持`get`和`init`方法。`init`是一个兼容`async`的方法，调用时，只含有一个参数：共享作用域对象。此对象在远程容器中用作共享作用于，并由host提供的模块填充。可以利用它在运行时动态地将远程容器连接到host容器。

### 公共路径
`publicPath`可以指定应用程序中所有资源的基础路径。

### 常用插件
+ `HtmlWebpackPlugin` 可以让改插件动态生成HTML文件

