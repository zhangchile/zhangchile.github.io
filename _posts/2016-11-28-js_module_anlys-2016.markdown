---
layout: post
title : 模块化加载标准的理解
subtitle: ""
date: 2016-11-28
author: "chile.zhang"
header-img: "img/post-bg-2016.jpg"
tags:
    - Javascript
---

## 模块化加载标准的理解

### AMD（Modules/AsynchronousDefinition）标准
在这个标准中，模块的加载都是提前异步加载的，模块中的变量都是包裹在一个函数define()中的，基本定义是这样的：

```javascript
define(string id?, array dependencies?, Object|Function factory);
```

其中id（模块标识），dependencies（依赖模块）是可选的参数，而factory是具体的功能函数或对象。
依赖会在执行factory前加载完成，factory是依赖加载后的回调，依赖的对象会依次注入到factory中。
一个完整的例子是这样的：

```javascript
define([dependence], function (dependence) {
  // use dependence obj to do something
  // dependence.foo()...
  return function () {
    console.log('hello');
  }
}
```

- 所有的依赖都是提前加载的，即便没有用到也会加载。
- 包裹在define里可以很好的避免全局污染

实现这个标准且用得比较多的加载器：  
[1].[RequireJS](http://requirejs.org/)  
[2].[curl.js](https://github.com/cujojs/curl/)  
[3].[Dojo](http://dojotoolkit.org/reference-guide/1.10/loader/)  

### Modules/Wrappings 标准
在这个规范中，模块加载是无需提前加载的，可以在模块的factory中按需加载的，这样可以大大的提高加载性能。

这个标准目的是为了解决浏览器端的模块加载问题，多数情况下浏览器加载js文件是通过XMLHttpRequest或者动态插入scrip标签来加载的，
XMLHttpRequest是限制了同源策略，而动态插入的会无法共享同一个上下文，所以有了这个浏览器端模块加载器解决方案。  
基本定义：

```javascript
module.declear(function (require, exports, module) {
  exports.foo = "foo";
});
```

Sea.js使用的是这个规范

### CommonJS规范
这个规范最重要的是加载的时候是同步的，
基本定义：

```javascript
var foo1 = require('foo1');
var foo2 = require('foo2');
var func = function () {
  return foo1 + foo2;
}
// ...
module.exports = func;
```

require是同步加载的，只有foo1加载完成了才会继续加载foo2模块，为的是解决服务端的模块加载问题

Node.js使用的是这个规范。

参考:  
[1].[Modules Wrappings](http://wiki.commonjs.org/wiki/Modules/Wrappings)  
[2].[Modules AsynchronousDefinition](http://wiki.commonjs.org/wiki/Modules/AsynchronousDefinition)  
[3].[common js](...)