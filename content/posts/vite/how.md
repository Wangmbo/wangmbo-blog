---
title: "如何使用vite"
date: 2021-08-13T09:51:32+08:00
draft: true
hiddenFromHomePage: true
categories: ["note"]
tags: ["vite"]
---
### 工程痛点
1. 模块化需求
2. 兼容浏览器
3. 线上代码质量
4. 开发效率

前端构建工具如何解决:
1. 模块化方案
   1. 提供模块加载方案
   2. 兼容不同模块规范
2. 语法转译
   1. 高级语法转译
   2. 资源加载
3. 产物质量
   1. 产物压缩、代码混淆、Tree Shaking
4. 开发效率
   1. HMR, 构建提速

### 模块化标准
#### 无标准阶段
1. 文件划分
   1. 无清晰依赖关系，全局变量冲突
2. 命名空间, 使用对象进行管理，避免了全局命名问题
   1. 无清晰依赖关系
3. IIFE私有作用域  实现私有成员功能
   1. 还是无法解决模块加载问题，如果存在依赖关系，script标签的加载顺序就要受到严格控制

#### CommonJS
模块规范一般包含两方面：
1. 统一的模块化代码规范
2. 实现自动加载模块的加载器

`require`导入，`module.exports`导出一个模块 

由Nodejs 提供，依赖了nodejs本身的功能实现，比如文件系统，如果CommonJS模块直接放到浏览器中是无法执行的.
CommonJS本身约定以同步的方式，进行模块加载，这种机制在服务端是没问题的。
1. 模块都在本地，不需要网络IO
2. 只有服务启动时才加载模块，服务通常启动后会一直运行

放在浏览器会有性能问题，浏览器要等待响应才能继续解析模块

#### AMD
异步模块定义规范

`define`定义模块，如果依赖其他模块则需要通过define的第一个参数来声明依赖，这样模块的代码执行之前浏览器慧先加载依赖模块。

`require`加载模块

CMD 与 AMD 类似

UMD 是兼容AMD和CommonJS 的一个模块化方案，可以运行在浏览器和Node.js环境。

1. 语法复相对复杂
2. 需要手动管理依赖
3. 原生不支持

#### ES6 Module
官方规范，Vite开发阶段 no-bundle的原因是由于模块加载的任务交给了浏览器，即使不打包也可以顺利运行模块代码。esModule能够同时在浏览器与Nodejs环境中执行，拥有天然的跨平台能力。



### 使用
```
<script type="module" src="/src/main.tsx"></script>
```
浏览器并不识别tsx语法，也无法直接import css 文件，上面的代码究竟是如何被浏览器正常执行呢。
这就归功于Vite Dev Server 所做的中间处理了，
在读取到main.tsx文件的内容后，Vite会对文件的内容进行编译。

利用浏览器原生ES模块的支持，实现开发阶段的Dev Server，进行模块的按需加载，而不是先整体打包再进行加载。


#### 初识配置文件
1. 通过命令行参数配置，如`vite --port=8888`
2. 通过配置文件配置

#### 生产环境构建
在开发阶段通过Dev Server 实现了不打包的特性，而在生产环境中，Vite依然会基于Rollup进行打包，并采取一系列的打包优化手段。


#### Css 工程化
Vite本身对Css各种预处理语言做了内置支持，也就是说及时不经过任何配置也可以使用css预处理器。

```
// vite.config.ts
import{normalizePath}from'vite';
//如果类型报错,需要安装@types/node:pnpmi@types/node-l0
import path from 'path';
全局 scss文件的路径
//用normalizePath解决 window 下的路径问题
const variablePath = normalizePath(path.resolve('./src/variable.scss'));
export default defineConfig({
//css相关的配置
CSS:{
preprocessorOptions:{
scss:{
//additionalData的内容会在每个 scss文件的开头自动动注入
additionalData: `@import "${variablePath}";
```

+ additionalData 会在每个scss文件的开头自动注入

##### CSS Modules 
CSS Modules 在Vite也是一个开箱即用的能力，Vite会对后缀带有`.module`的样式文件自动应用CSS Modules。
1. 预处理
2. CSS Modules
3. css in js, styled-component
4. tailwind windi


#### Lint
+ Eslint 是在ECMAScript/JavaScript 代码中识别和报告模式匹配的工具，它的目标是保证代码的一致性和避免错误
为了打造一款插件化的JavaScript代码静态检查工具，通过解析代码的AST来分析代码格式，检查代码的风格和质量问题。
ESLint的使用并不复杂，主要通过配置文件对各种代码格式的规则进行配置。
```
pnpm i eslint -D
// eslint 初始化
npx eslint --init
```
+ 添加插件后只是拓展了ESLint 本身的规则集，但ESLint默认并没有开启这些规则的校验。如果要开启或者调整这些规则，你需要在rules中配置。
+ env和globals
  + 这两个配置分别表示运行环境和全局环境，指定运行环境会预设一些全局变量，例如
  ```
  module.export = {
    "env": {
      "browser": "true",
      "node": "true"
    }
  }
  ```
  指定上述的`env`配置后便会启用浏览器和Node.js环境，这两个环境中的一些全局变量会同时启用

  有些全局变量是业务代码引入的第三方库所声明，这里就需要在`globals`配置中声明全局变量了。
  ```
  module.export = {
    "globals": {
      // 不可重写
      "$": false,
      "jQuery": false
    }
  }
  ```

##### 与Prettier强强联合
虽然ESLint本身具备自动格式化代码的功能(eslint --fix)，但是术业有专攻，ESLint的主要优势在于`代码的风格检查并给出提示`，而在代码格式化这一块，Prettier做的更加专业。因此我们经常讲Eslint结合Prettier一起使用

通过配置
.eslintrc.js
.prettierrc.js

在VSCode中安装ESLint和Prettier插件，然后在VSCode的配置文件中添加如下配置
```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```
在保存时就可以自动修复代码格式

##### 样式规范工具 Stylelint

##### Husky + lint-staged的Git提交工作流集成
我们安装`ESlint`、`Prettier`和`Stylelint`的VSCode插件或者Vite插件，在开发阶段提前规避掉代码格式问题，但实际上也只是将问题提前暴露,并不能保证线上代码出现不符合规范的情况。
我们可以在代码提交时进行卡点检查，也就是拦截git commit 命令，进行代码格式检查，只有确保通过格式检查才允许正常提交代码。
Git提交规范`Commitlint`

`Husky` + `lint-staged` 拦截`git commit` 过程，只有各项Lint检查通过后才能正常提交代码.

1. 初始化Husky
```json
{
 "scripts": {
 // 会在安装 npm 依赖后自动执行
 "postinstall": "husky install"
 }
}
```

2. 添加Husky钩子
npx husky add .husky/pre-commit "npm run lint"

接着你将会在项目根目录的 .husky 目录中看到名为 pre-commit 的文件，里面包含
了 git commit 前要执行的脚本。现在，当你执行 git commit 的时候，会首先执行 npm
run lint 脚本，通过 Lint 检查后才会正式提交代码记录。
不过，刚才我们直接在 Husky 的钩子中执行 npm run lint ，这会产生一个额外的问题:
Husky 中每次执行 npm run lint 都对仓库中的代码进行全量检查，也就是说，即使某些
文件并没有改动，也会走一次 Lint 检查，当项目代码越来越多的时候，提交的过程会越
来越慢，影响开发体验。


而 `lint-staged` 就是用来解决上述全量扫描问题的，可以实现只对存入 暂存区 的文件进
行 Lint 检查，大大提高了提交代码的效率。首先，让我们安装一下对应的 npm 包:

然后在 package.json 中添加如下的配置:
```json
{
 "lint-staged": {
 "**/*.{js,jsx,tsx,ts}": [
 "npm run lint:script",
 "git add ."
 ],
 "**/*.{scss}": [
 "npm run lint:style",
 "git add ."
 ]
 }
}
```

接下来我们需要在 Husky 中应用 lint-stage ，回到 .husky/pre-commit 脚本中，将原来
的 npm run lint 换成如下脚本:
npx --no -- lint-staged
如此一来，我们便实现了提交代码时的 增量 Lint 检查 。


#### 静态资源: 如何在Vite中处理各种静态资源
一方面我们需要解决资源加载的问题，对于Vite来说就是如何将静态资源解析并加载为一个ES模块的问题；
另一方面在生产环境下我们还需要考虑静态资源的部署问题、
体积问题、网络性能问题，并采取相应的方案来进行优化。

1. 使用场景
   1. img src 
   2. background
   3. 脚本动态加载

2. 引用方式

```js
import logoSrc from '@assets/imgs/vite.png'
export function Header() {
 return (
 <div className={`p-20px text-center ${styles.header}`}>
 <!-- 省略前面的组件内容 -->
 <!-- 使用图片 -->
 <img className="m-auto mb-4" src={logoSrc} alt="" />
 </div>
 );
}
```
3. SVG组件加载方式

4. Work脚本
然后在 Header 组件中引入，引入的时候注意加上 ?worker 后缀，相当于告诉 Vite 这是
一个 Web Worker 脚本文件
```js
import Worker from './example.js?worker';
// 1. 初始化 Worker 实例
const worker = new Worker();
// 2. 主线程监听 worker 的信息
worker.addEventListener('message', (e) => {
 console.log(e);
});
```

##### 生产环境处理
1. 部署域名怎么配置 => 根据环境变量更换base路径
2. 资源打包成单文件还是作为 Base64 格式内联?
   在Vite中，所有的静态资源都有两种构建方式，一种是打包成单文件，另一种是通过base64编码的格式内嵌到代码中。
   小文件建议base64
   大文件啊还是提取成单独文件


3. 图片太大了怎么压缩？
   vite-plugin-imagemin 压缩图片
4. svg 请求数量太多了怎么优化？
    雪碧图优化
    雪碧图包含了所有图标的具体内容，对于页面每个具体的图标，则通过use属性来引用雪碧图的对应内容


#### 如何玩转秒级依赖预构建能力
所谓的`no-bundle`只是对于源代码而言，对于第三方依赖Vite还是选择bundle，并且使用极快的打包器Esbuild来完成这一过程。

1. 为什么需要预构建
   1. 我们没办法控制第三方的打包规范
   2. 请求瀑布流，第三方依赖设计模块数量多，会触发成百上千发个请求。

依赖预构建做了两件事：
1. 其他格式转为ESM
2. 打包第三方库，将分散文件合并在一起

由`ESbuild`完成

#### 自定义配置详解
入口文件 entries

include 强制预构建配置
我们要尽力避免运行时的二次预构建，可以通过`include`参数提前声明需要按需加载的依赖。

自定义 Esbuild 行为


#### ESbuild
esbuild在vite中扮演的角色：
1. 依赖预构建
2. Ts/jsx编译
3. 压缩代码

为什么性能好：
1. Golang开发
2. 多核并行
3. 无第三方库
4. 高效的内存利用，从头到尾尽可能地复用一份AST节点数据

项目打包-build API
单文件转译-Transform API
Esbuild插件开发

#### Rollup
对于一次完整的构建过程而言，Rollup回显进入到Build阶段，解析各模块的内容及依赖关系，然后进入`Output`阶段，
完成打包及输出的过程。

实际上插件的各种Hook可以根据这两个构建阶段分为两类`Build Hook`与`Output Hook`。
+ `Build Hook`即在`Build`阶段执行的钩子函数，在这个阶段主要进行模块代码的转换，AST解析以及模块依赖的解析，那么这个阶段的Hook对于代码的操作粒度一般为模块级别，也就是单文件级别。
+ `Output Hook` ，则主要进行代码的打包，对于代码而言，操作粒度一般是chunk级别。

同步 异步 并行 串行 First

Build Hooks的工作流程
1. 首先经历options钩子进行配置的转换，得到处理后的配置对象
2. buildStart钩子，开始构建，可以进行一些初始化操作
3. 进入到resolveId钩子中解析文件路径，（从input）配置指定的入口文件开始
4. Rollup通过调用load钩子加载模块内容
5. 执行所有的transform 钩子对模块内容进行自定义转换，比如babel转译
6. Rollup拿到最后的模块内容，进行AST分析，得到所有的import内容，调用moduleParsed钩子：
   1. 如果是普通的import，则执行resolveId钩子，继续回到步骤3
   2. 如果是动态import，则执行 resolveDynamicImport 钩子解析路径，如果解析成功，则回到4加载模块，否则回到步骤3通过resolveId解析路径。
7. 直到所有import都解析完毕，Rollup执行buildEnd钩子，Build阶段结束。

Output 阶段工作流
1. 执行所有插件的outputOptions钩子函数，对output配置进行转换
2. 执行renderStart并发执行renderStart钩子，正式开始打包
3. 对每个即将生成的chunk执行augmentChunkHash钩子，来决定是否更改chunk的哈希值
4. 如果没有遇到import.meta语句则进入下一步，否则调用resolveFileUrl或resolveImportMeta来进行自定义解析
5. 接着 Rollup 会生成所有 chunk 的内容，针对每个 chunk 会依次调用插件的renderChunk 方法进行自定义操作，也就是说，在这里时候你可以直接操作打包产物了
6. 最后调用generateBundle, 入参为chunk ,asset。在这里可以删除chunk或者asset
7. generateBubdle => 输出产物到磁盘 => writeBundle


插件机制： 将核心逻辑和场景处理逻辑分离，可以按需引入插件功能


#### Vite插件介绍
通用hook

1. 服务器启动阶段： options和buildStart钩子会在服务启动时被调用
2. 请求响应阶段：当浏览器发起请求时，Vite内部依次调用`resolvedId`、`load`和`transform`钩子
3. 服务器关闭阶段：Vite会依次执行`buildEnd`和`closeBundle`钩子

除了以上钩子，其他Rollup的钩子均不在Vite开发阶段调用


通用Hook
1. config，返回一个配置对象，这个配置对象会和vite已有的配置进行深度的合并
2. configResolved，vite解析完配置后会调用
3. configureServer 获取dev实例
4. transformIndexHtml
5. handleHotUpdate 热更新处理


Vite插件执行顺序：
1. Alias相关插件
2. 带有`enfrce: 'pre'`的用户插件
3. Vite核心插件
4. 普通插件
5. 生产插件
6. 带有`enfrce: 'post'`的用户插件
7. Vite后置构建插件，例如压缩插件

#### 实现插件
+ 虚拟模块
```js
import { Plugin } from 'vite'
const virtualFibModuleId = 'virtual:fib'
// Vite 中约定对于虚拟模块，解析后的路径需要加上`\0`前缀
const resolvedFibVirtualModuleId = '\0' + virtualFibModuleId;

export default function virtualFibModulePlugin(): Plugin {
  // const config: ResolvedConfig | null = null;
  return {
    name: 'vite-plugin-virtual-module',

    resolveId(id) {
      if (id === virtualFibModuleId) { 
        return resolvedFibVirtualModuleId;
      }
    },
    load(id) {
      // 加载虚拟模块
      if (id === resolvedFibVirtualModuleId) {
        return 'export default function fib(n) { return n <= 1 ? n : fib(n - 1) + fib(n - 2) };'
      }
    }
  }
 }
```
不存在磁盘而在内存当中也就是虚拟模块。我们既可以把自己手写的代码字符串作为单独的模块内容，又可以将内存中某些经过计算得出的变量作为模块内容进行加载，非常灵活和方便。


`vite-plugin-inspect`插件可以用于调试插件


#### HMR 简介
HMR全称叫做`Hot Module Replacement` 即模块热替换
页面模块更新时，把页面中发生的模块替换成新的模块

##### import.meta.hot.accept
1. 接受自身模块更新
2. 某个子模块更新
3. 接受多个子模块的更新

1. 接受自身模块更新
```js
if(import.meta.hot) {
  import.meta.hot.accept((mod) => mod.render())
}
```
2. 接受依赖模块更新
```js
if (import.meta.hot) {
  import.meta.hot.accept('./render.ts', (newModule) => {
    newModule.render();
  })
}
```
3. 接受多个子模块的更新
```js
if (import.meta.hot) {
 import.meta.hot.accept(['./render.ts', './state.ts'], (modules) => {
 // 自定义更新
 const [renderModule, stateModule] = modules;
 if (renderModule) {
 renderModule.render();
 }
 if (stateModule) {
 stateModule.initState();
 }
 })
}
```

`hot.dispose`
在模块更新、旧模块需要销毁做一些事情
```js
if (import.meta.hot) {
 import.meta.hot.dispose(() => {
 if (timer) {
 clearInterval(timer);
 }
 })
}
```
但是这样之前的状态没有了

`hot.data`这个属性在不同模块实例之间共享了一些数据。

1. import.meta.hot.decline()
这个方法调用之后，相当于表示此模块不可热更新，当模块更新时会强制进行页面刷新。
感兴趣的同学可以继续拿上面的例子来尝试一下。
2. import.meta.hot.invalidate()
这个方法就更简单了，只是用来强制刷新页面。


HMR 模块局部更新和状态保存
可以看到代码发生变动的时候，Vite会定位到发生变化的局部模块，也就是找到对应HMR边界，然后基于这个边界进行更新，其他模块并没有受到影响，这也是Vite中热更新的时间也到达毫秒级别的原因。

#### 代码分割
`bundle`整体打包产物
`chunk`打包后的JS文件，bundle的子集
`vendor`第三方打包的产物

单chunk打包模式缺点：
1. 无法按需加载
2. 缓存复用率极低

一方面，Vite实现了CSS代码分割的能力，即实现了一个chunk对应一个css文件，比如上面产物中`index.js`对应一份`index.css`，而按需加载的chunk `Danamic.js`也对应一份单独的Danamic.css文件，与JS文件的代码分割同理，这样做也能提升CSS文件的缓存复用率。

而另一方面，Vite基于Rollup的`manualChunks`API实现了`应用拆包`的策略：
1. initial chunk 单独为一个index.js
2. 对于 Async chunk ，动态import的代码会被拆分成单独的chunk

