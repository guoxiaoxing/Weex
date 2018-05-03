# Weex for Android原理分析：原理概述

作者：[郭孝星](https://github.com/guoxiaoxing)

校对：[郭孝星](https://github.com/guoxiaoxing)

文章状态：编辑中

**关于项目**

> [Weex](https://github.com/guoxiaoxing/Weex)项目分析关于Weex的原理分析与最佳实践。

**文章目录**

- 官方网站：http://weex.apache.org/cn/

本篇文章开始我们来分析Weex的原理实现。

Weex源码目录如下所示：

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_source_code_structure.png" width="500">

Weex的设计理念

- 性能为王：Weex会将JS组件渲染成原生的UI，达到最佳的性能和用户体验，Weex还采用多种手段优化性能表现，包括优化 native 与 JavaScript 的通信频率和通信量，使用二进制方式提升单次通信的耗时等，未来还会通过跨平台内核将 DOM 管理移至 native 层实现来彻底解决 native 与 JavaScript 层异步通信成本的问题，从多个维度提升 Weex 引擎的性能。
- 交互体验：Weex希望完全做到iOS、Android与Web真正的跨度，一套代码，三端运行。
- 开发效率：利用既有的前端技术栈，从易用和高效两个角度提升Weex的开发体验。
- 易于扩展：Weex可以利用Module扩展原生功能，利用Component扩展原生UI功能。

讲完了Weex的设计理念，我们再来看看Weex的实现原理。

Weex运行原理如下所示：

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_principle.png" width="500">

1. Weex会将JavaScript代码生成一个Weex的JS Bundle。
2. 开发者可以将生成的 JS bundle 部署至云端，然后通过网络请求或预下发的方式加载至用户的移动应用客户端。
3. 在移动应用客户端里，Weex SDK 会准备好一个 JavaScript 执行环境，并且在用户打开一个 Weex 页面时在这个执行环境中执行
   相应的 JS bundle，并将执行过程中产生的各种命令发送到 native 端进行界面渲染、数据存储、网络通信、调用设备功能及用户交互响应等功能；同时，如果
   用户希望使用浏览器访问这个界面，那么他可以在浏览器里打开一个相同的 web 页面，这个页面和移动应用使用相同的页面源代码，但被编译成适合Web展示的JS 
   Bundle，通过浏览器里的 JavaScript 引擎及 Weex SDK 运行起来的。
   
注：JavaScript执行环境指的是JavaScriptCore。