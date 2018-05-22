# Weex for Android原理分析：渲染原理

作者：[郭孝星](https://github.com/guoxiaoxing)

校对：[郭孝星](https://github.com/guoxiaoxing)

文章状态：编辑中

**关于项目**

> Weex 是一款轻量级的移动端跨平台动态性技术解决方案。

> [Weex](https://github.com/guoxiaoxing/Weex)项目分析关于Weex的原理分析与最佳实践。

**文章目录**

同步机制: JS Framework一次性创建所有的DOM树, 完成渲染.

异步机制

- 流式渲染: JS Framework每次创建一个节点, 立即发送给Native, 同时等待Native的next tick命令触发时才能继续执行, 这种
策略有效的解决了页面回退仍在渲染, 事件不能响应的问题.
- 离屏渲染: 在用户浏览页面的过程中, 后台DOM线程继续渲染更新页面, 将所有addDom()、updateSytle()等操作都以最小颗粒度拆分，保证
所有的操作都在16ms内完成，大大提升首屏的加载性能以及滑动的流畅度。


