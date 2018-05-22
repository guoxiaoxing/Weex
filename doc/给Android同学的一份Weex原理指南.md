# 给Android同学的一份Weex原理分析指南

作者：[郭孝星](https://github.com/guoxiaoxing)

校对：[郭孝星](https://github.com/guoxiaoxing)

文章状态：编辑中

**关于项目**

> [Weex](https://github.com/guoxiaoxing/Weex)项目分析关于Weex的原理分析与最佳实践。

**文章目录**

- 一 架构分析
- 二 启动流程
- 三 渲染机制
- 四 线程模型
- 五 通信机制


技术从来都是在业务的发展过程中, 不断演变而来的. React Native 与 Weex这样的框架也正是基于对业务动态化以及减少double人力的
诉求而诞生的.

先附上前端大佬同时也是Weex技术方案的创立者[@勾股](https://github.com/Jinjiang)的关于无线电商动态化的三篇文章:

- [对无线电商动态化方案的思考（一）](https://github.com/amfe/article/issues/13)
- [对无线电商动态化方案的思考（二）](https://github.com/amfe/article/issues/14)
- [对无线电商动态化方案的思考（三）](https://github.com/amfe/article/issues/15)

这三篇文章描述了Weex的起源和对整个技术方案的思考, 大家可以先看一下, 有助于我们理解后续的内容.

接下来, 我们来正式介绍Weex.

第一个问题, 什么是Weex?

> Weex 是一款轻量级的移动端跨平台动态性技术解决方案！

第二个问题, 和React Native相比, 它有什么区别与优势?

> React Native: Learn once, write anywhere.  Weex: Write once, run anywhere.

注: 更多关于Weex与React Native的对比大家可以参考这篇文章[weex vs react-native](https://yq.aliyun.com/articles/57996).

## 一 架构分析

Weex整体源码与Weex Android SDK源码目录如下所示：

<p>
<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_source_code.png" height="600">
<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_android_source_code.png" height="600">
<p/>

Weex架构设计如下所示:

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_structure.png"/>

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


<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_principle.png"/>


整个流程如下所示:

① Weex会通过Weex DSL将we文件解析成一个标准的JS文件, 并将这些JS文件打包成JS Bundle.
② 开发者可以将生成的 JS bundle 部署至云端，然后通过网络请求或预下发的方式加载至用户的移动应用客户端。
③ 在移动应用客户端里，Weex SDK 会准备好一个 JavaScript 执行环境，并且在用户打开一个 Weex 页面时在这个执行环境中执行相应的 JS bundle，并将执行过程中产生的各种命令发送到 native 端进行界面渲染、数据存储、网络通信、调用设备功能及用户交互响应等功能；同时，如果用户希望使用浏览器访问这个界面，那么他可以在浏览器里打开一个相同的 web 页面，这个页面和移动应用使用相同的页面源代码，但被编译成适合Web展示的JS Bundle，通过浏览器里的 JavaScript 引擎及 Weex SDK 运行起来的。
   
注：JavaScript执行环境指的是JavaScriptCore。

## 二 启动流程

每一套框架的使用三部曲就是: ① 集成依赖 ② 初始化 ③ 使用框架. 我们来看看在使用Weex之前, 做的初始化. 当你使用weex init 命令生成一个Demo工程, 其中的Application里
会有这么一段初始化代码:

```java
public class WXApplication extends Application {

  @Override
  public void onCreate() {
    super.onCreate();
    WXSDKEngine.addCustomOptions("appName", "WXSample");
    WXSDKEngine.addCustomOptions("appGroup", "WXApp");
    // 初始化Weex SDK
    WXSDKEngine.initialize(this,
        new InitConfig.Builder().setImgAdapter(new ImageAdapter()).build()
    );
    try {
      WXSDKEngine.registerModule("event", WXEventModule.class);
    } catch (WXException e) {
      e.printStackTrace();
    }
    // 初始化应用配置
    AppConfig.init(this);
    // 载入插件
    WeexPluginContainer.loadAll(this);
  }
}
```

其中最主要的就是WXSDKEngine.initialize()方法, 这个方法里有个初始化接口, 用来自定义我们自己的图片加载, 网络请求, 埋点等功能.

- private IWXHttpAdapter httpAdapter;// 网络请求
- private IDrawableLoader drawableLoader;// Drawable加载
- private IWXImgLoaderAdapter imgAdapter;// 图片加载
- private IWXUserTrackAdapter utAdapter;// 用户日志与埋点
- private IWXDebugAdapter debugAdapter;// debug调试
- private IWXStorageAdapter storageAdapter;// so加载
- private IWXSoLoaderAdapter soLoader;// 存储策略, 默认是Android的SQLite
- private URIAdapter mURIAdapter;// URI解析
- private IWebSocketAdapterFactory webSocketAdapterFactory;// WebSocket协议实现
- private IWXJSExceptionAdapter mJSExceptionAdapter;// 异常处理
- private String framework;


WXSDKEngine.initialize()方法会继续调用WXSDKEngine.doInitInternal()来完成初始化操作, 我们来具体看看它都做了什么.


```java
public class WXSDKEngine {

    private static void doInitInternal(final Application application,final InitConfig config){
        WXEnvironment.sApplication = application;
        WXEnvironment.JsFrameworkInit = false;

        WXBridgeManager.getInstance().post(new Runnable() {
          @Override
          public void run() {
            long start = System.currentTimeMillis();
            // 获取WXSDKManager单例, WXSDKManager是Weex上下文的管理类, 是个上帝类.
            WXSDKManager sm = WXSDKManager.getInstance();
            // 回调onSDKEngineInitialize()方法.
            sm.onSDKEngineInitialize();
            if(config != null ) {
              sm.setInitConfig(config);
              if(config.getDebugAdapter()!=null){
                config.getDebugAdapter().initDebug(application);
              }
            }
            // 初始化WXSoInstallMgrSdk, WXSoInstallMgrSdk用来加载so库与管理so库版本.
            WXSoInstallMgrSdk.init(application,
                                  sm.getIWXSoLoaderAdapter(),
                                  sm.getWXStatisticsListener());
            // 加载so库,这个V8_SO_NAME为"weexjsc"
            boolean isSoInitSuccess = WXSoInstallMgrSdk.initSo(V8_SO_NAME, 1, config!=null?config.getUtAdapter():null);
            if (!isSoInitSuccess) {
              return;
            }
            // 初始化JS Framework, 这个会通过WXBridgeManager向JS发送一个执行初始化的消息.
            sm.initScriptsFramework(config!=null?config.getFramework():null);

            WXEnvironment.sSDKInitExecuteTime = System.currentTimeMillis() - start;
            WXLogUtils.renderPerformanceLog("SDKInitExecuteTime", WXEnvironment.sSDKInitExecuteTime);
          }
        });

        // 注册Component, Module与DomObject
        register();
      }

}
```

从上面可以看出, 整个Weex SDK初始化的过程主要做了以下几件事:

1. 获取WXSDKManager单例, WXSDKManager是Weex上下文的管理类, 是个上帝类.
2. 初始化WXSoInstallMgrSdk, WXSoInstallMgrSdk用来加载so库与管理so库版本.
3. 加载so库,这个V8_SO_NAME为"weexjsc".
4. 初始化JS Framework, 这个会通过WXBridgeManager向JS发送一个执行初始化的消息.
5. 注册Component, Module与WXDomObject.

最后一步中, 提到了几个概念, 我们来简单了解一下.

什么是Module？

> Module 是完成一个操作的方法集合，在 Weex 的页面中，允许开发者 require 引入，调用 module 中的方法，WeexSDK 在启动时候，已经注册了一些内置的 module。
负责管理Module的是WXModuleManager, 该类完成了Module的注册与调用.

什么是Component?

> Component描述了扩展的Native UI信息, 用来提供给JS使用.

什么是WXDomObject?

> WXDomObject用来描述Dom节点, 包含了Dom节点的所有信息, 包含style, attribute and event.

了解了Weex SDK的启动流程, 我们再看看看Weex页面是如何渲染最终显示到Android设备上的.

## 三 渲染机制

老样子, 我们先不讲各种理论, 从我们能接触到细节入手开始分析.

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_render_sequence.png"/>

整个渲染流程可以概括为:

① createBody/addDom完成从Dom Tree到Component Tree的映射.
② layout操作, 有batch驱动, 每隔16ms执行一批任务, 开始渲染.
③ 生成的root component会被添加到Render Component.

Dom节点的数据是用json格式来描述的, 如下所示:

```
{
    "attr":{"spmId":"spma"},
    "ref":"_root",
    "style":{},
    "type":"div"
}
```


同步机制: JS Framework一次性创建所有的DOM树, 完成渲染.

异步机制

- 流式渲染: JS Framework每次创建一个节点, 立即发送给Native, 同时等待Native的next tick命令触发时才能继续执行, 这种
策略有效的解决了页面回退仍在渲染, 事件不能响应的问题.
- 离屏渲染: 在用户浏览页面的过程中, 后台DOM线程继续渲染更新页面, 将所有addDom()、updateSytle()等操作都以最小颗粒度拆分，保证
所有的操作都在16ms内完成，大大提升首屏的加载性能以及滑动的流畅度。


## 四 线程模型

Weex里有三大线程, 线程模型序列图如下所示:

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_thread_sequence.png"/>

- JSBridgeThread: 用来进行Java JNI层与V8 Engine之间的通信, 同时还负责初始化JS Framework, 调用JS, 调用Native.
- UIThread: 用来操作与渲染视图, 数据绑定等.
- DomThread: 用来执行Dom操作, 包括Dom解析, 设置Dom样式, CSS Layout操作, 生成Component Tree.

注: Weex里的线程使用的是封装类WXThread, WXThread继承于HandlerThread, 本质上是一个带消息循环的线程.

## 通信机制

线程间的通信都是通过Android里的Handler机制来完成的, 因此线程中的所有操作都是时序性的, Dom的操作也是时序性的. 如下所示:

```java
Message msg = Message.obtain();
WXDomTask task = new WXDomTask();
…
msg.what = WXDomHandler.MsgType.WX_DOM_CREATE_BODY;
msg.obj = task;
WXSDKManager.getInstance().getWXDomManager().sendMessage(msg);
```

- UIThread 与 JSBridgeThread： JSBridgeThread 不会直接发送任务给 UIThread ， UIThread 发送给 JSBridgeThread 的任务有初始化js framework、开始渲染页面createInstance、发送event事件等。
- UIThread 与 DomThread： UIThread 会在销毁instance的时候发送任务给 DomThread 进行清理，DomThread 发送任务给 UIThread 会分为两步，这两步会是一个task：发送前会重新计算CSSLayout的耗时操作，这部分的操作是在DomThread中进行。
发送 runnable 到 UIThread，runnable执行的就是view的渲染流程，在UIThread中进行。说明： 这一整个task是每隔16ms自动触发，也是说一旦dom操作过多，就会拖累帧率。
- JSBridgeThread 与 DomThread：DomThread不会直接发送任务给JSBridgeThread 。js runtime会通过jni发送指令到 java 层，这一部分在JSBridgeThread中，然后JSBridgeThread会发送任务给 DomThread 进行各种 Dom 操作。

## 附录

相关资源

官方网站

- [weexteam](https://github.com/weexteam)
- [官方网站](https://weex.apache.org/cn/)
- [讨论组](https://github.com/weexteam/article/issues)

集成指南

- [Android&iOS集成指南](https://weex.incubator.apache.org/cn/guide/integrate-to-your-app.html)
- [H5集成指南](https://github.com/weexteam/article/issues/10)

其他资源

- [weex-hackernews](https://github.com/weexteam/weex-hackernews)
- [weex-vue-examples](https://github.com/Hanks10100/weex-vue-examples)
- [Vue.js](https://cn.vuejs.org/)

为了方便没有前端基础的客户端工程师如何想开始学习Weex, 这边也给大家列了一份Weex技术体系图.

<img src="https://github.com/guoxiaoxing/Weex/raw/master/art/principle/weex_system.png">