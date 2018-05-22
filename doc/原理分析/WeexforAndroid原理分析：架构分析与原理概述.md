# WeexForAndroid原理分析：原理概述

作者：[郭孝星](https://github.com/guoxiaoxing)

校对：[郭孝星](https://github.com/guoxiaoxing)

文章状态：编辑中

**关于项目**

> [Weex](https://github.com/guoxiaoxing/Weex)项目分析关于Weex的原理分析与最佳实践。

**文章目录**

技术从来都是在业务的发展过程中, 不断演变而来的. React Native 与 Weex这样的框架也正是基于对业务动态化以及减少double人力的
诉求而诞生的.

先附上前端大佬同时也是Weex技术方案的创立者[@勾股](https://github.com/Jinjiang)的关于无线电商动态化的三篇文章:

- [对无线电商动态化方案的思考（一）]https://github.com/amfe/article/issues/13)
- [对无线电商动态化方案的思考（二）](https://github.com/amfe/article/issues/14)
- [对无线电商动态化方案的思考（三）](https://github.com/amfe/article/issues/15)

这三篇文章描述了Weex的起源和对整个技术方案的思考, 大家可以先看一下, 有助于我们理解后续的内容.

接下来, 我们来正式介绍Weex.

第一个问题, 什么是Weex?

> Weex 是一款轻量级的移动端跨平台动态性技术解决方案！

第二个问题, 和React Native相比, 它有什么区别与优势?

> React Native: Learn once, write anywhere.  Weex: Write once, run anywhere.

注: 更多关于Weex与React Native的对比大家可以参考这篇文章[weex vs react-native](https://yq.aliyun.com/articles/57996).

Weex源码目录如下所示：

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_source_code.png">

Weex Android SDK的源码目录如下所示:

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_android_source_code.png">

Weex架构设计如下所示:

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_source_code_structure.png" width="500">

我们来简单介绍下各个层次的作用.

> JavaScript Framework层主要完成了Weex DSL

- Weex DSL: Weex DSL对常用的页面组件进行了封装, 并且暴露规范化的标签\特性\样式\事件与上下级约束的定义, 所有的业务页面都可以
基于这些基础组件搭建而成.Weex DSL基于Vue.js, 将一个组件分成把一个组件分成 <template>、<style>、<script> 三部分，分别表达
一个组件的界面结构、界面样式、数据&逻辑.
- JS Framework: JS Framework事实上就是一个js文件main.js, 它负责JS与Native之间的交互, 数据绑定与事件逻辑处理等工作.

> C++ Framework层主要就是JavaScriptCore/V8的so库, 用来解析执行JS, 以及连接Java与JavaScript.

> Java Framework层主要封装了双向通信, Native View渲染, Dom Tree生成与修改等功能.

- JS Bridge: JS Bridge用来与JS Engine(V8)进行双向通信, 运行在JS Bridge线程中. Weex初始化, Component, Module, DomBject
的的注册于调用, JS Bridge线程的管理最终会交由WXBridgeManager来完成.
- Render: Render主要负责渲染Native View, 运行在UI线程中, 由WxRenderManager统一管理, 具体操作由WxRenderStatement来完成.
- Dom: Dom主要用来操作Dom结构, 生成对应的Dom Tree. 运行在Dom线程中, 由WXDomManager统一管理.

聊完了Weex的架构设计, 我们再简单来看一下它的运行原理, 让大家有个整体的印象. Weex运行原理如下所示：

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_principle.png" width="500">

1. Weex会将JavaScript代码生成一个Weex的JS Bundle。
2. 开发者可以将生成的 JS bundle 部署至云端，然后通过网络请求或预下发的方式加载至用户的移动应用客户端。
3. 在移动应用客户端里，Weex SDK 会准备好一个 JavaScript 执行环境，并且在用户打开一个 Weex 页面时在这个执行环境中执行
   相应的 JS bundle，并将执行过程中产生的各种命令发送到 native 端进行界面渲染、数据存储、网络通信、调用设备功能及用户交互响应等功能；同时，如果
   用户希望使用浏览器访问这个界面，那么他可以在浏览器里打开一个相同的 web 页面，这个页面和移动应用使用相同的页面源代码，但被编译成适合Web展示的JS 
   Bundle，通过浏览器里的 JavaScript 引擎及 Weex SDK 运行起来的。
   
注：JavaScript执行环境指的是JavaScriptCore。

## 附录

相关资源

官方网站

- [官方网站](https://weex.apache.org/cn/)
- [讨论组](https://github.com/weexteam/article/issues)

集成指南

- [Android&iOS集成指南](https://weex.incubator.apache.org/cn/guide/integrate-to-your-app.html)
- [H5集成指南](https://github.com/weexteam/article/issues/10)

为了方便没有前端基础的客户端工程师如何想开始学习Weex, 这边也给大家列了一份Weex技术图谱.

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_system.png">