# Weex for Android原理分析：原理概述

作者：[郭孝星](https://github.com/guoxiaoxing)

校对：[郭孝星](https://github.com/guoxiaoxing)

文章状态：编辑中

**关于项目**

> [Weex](https://github.com/guoxiaoxing/Weex)项目分析关于Weex的原理分析与最佳实践。

**文章目录**

什么是Module？

> module 是完成一个操作的方法集合，在 Weex 的页面中，允许开发者 require 引入，调用 module 中的方法，WeexSDK 在启动时候，已经注册了一些内置的 module。

我们可以自定义Module，Weex也内置了很多Module，例如网络请求Module：

```javascrpt
var stream = weex.requireModule('stream');
stream.fetch({
       method: 'GET',
       url: 'http://httpbin.org/get',
       type:'jsonp'
     }, function(ret) {
	  console.log('in completion')
     },function(response){
       console.log('in progress')
     });
```

我们来分析一下Module的注册雨调用流程。


## 一 Module注册流程

在Weex中Module也分成两种：

- 实例Module：这种Module每次创建时都会创建一个实例。
- 全局Module：这种Module是单例的，每个WXSDKEngine创建时都只会创建一次。


> WXModuleManager用来管理所有的Module。

WXModuleManager内部有四个map，分别用来存储不同的Module。如下所示：

```java
public class WXModuleManager {
      /**
       * module class object dictionary
       */  
      private static volatile ConcurrentMap<String, ModuleFactoryImpl> sModuleFactoryMap = new ConcurrentHashMap<>();
      private static Map<String, WXModule> sGlobalModuleMap = new HashMap<>();
      private static Map<String, WXDomModule> sDomModuleMap = new HashMap<>();
    
      /**
       * module object dictionary
       * K : instanceId, V : Modules
       */
      private static Map<String, Map<String, WXModule>> sInstanceModuleMap = new ConcurrentHashMap<>();
}
```



```java
public class WXModuleManager {

}
```