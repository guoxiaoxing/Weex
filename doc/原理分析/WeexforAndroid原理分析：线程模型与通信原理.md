# Weex for Android原理分析：线程模型

作者：[郭孝星](https://github.com/guoxiaoxing)

校对：[郭孝星](https://github.com/guoxiaoxing)

文章状态：编辑中

**关于项目**

> [Weex](https://github.com/guoxiaoxing/Weex)项目分析关于Weex的原理分析与最佳实践。

**文章目录**


Weex里有三大线程:

- JSBridgeThread: 用来进行Java JNI层与V8 Engine之间的通信, 同时还负责初始化JS Framework, 调用JS, 调用Native.
- UIThread: 用来操作与渲染视图, 数据绑定等.
- DomThread: 用来执行Dom操作, 包括Dom解析, 设置Dom样式, CSS Layout操作, 生成Component Tree.

线程间的通信都是通过Android里的Handler机制来完成的, 因此线程中的所有操作都是时序性的, Dom的操作也是时序性的.

- UIThread 与 JSBridgeThread： JSBridgeThread 不会直接发送任务给 UIThread ， UIThread 发送给 JSBridgeThread 的任务有初始化js framework、开始渲染页面createInstance、发送event事件等。
- UIThread 与 DomThread： UIThread 会在销毁instance的时候发送任务给 DomThread 进行清理，DomThread 发送任务给 UIThread 会分为两步，这两步会是一个task：发送前会重新计算CSSLayout的耗时操作，这部分的操作是在DomThread中进行。
发送 runnable 到 UIThread，runnable执行的就是view的渲染流程，在UIThread中进行。说明： 这一整个task是每隔16ms自动触发，也是说一旦dom操作过多，就会拖累帧率。
- JSBridgeThread 与 DomThread：DomThread不会直接发送任务给JSBridgeThread 。js runtime会通过jni发送指令到 java 层，这一部分在JSBridgeThread中，然后JSBridgeThread会发送任务给 DomThread 进行各种 Dom 操作。