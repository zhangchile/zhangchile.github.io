---
layout: post
title : 模块加载器的一些理解
subtitle: ""
date: 2016-12-06
author: "chile.zhang"
header-img: "img/post-bg-2016.jpg"
tags:
    - Javascript  
    - 模块化  
    - 前端  
---

## 模块加载器的一些理解
在日常工作中接触到一些模块加载，对比了一下，随便写写。

### Sea.js

**factory执行的时机**  
seajs是玉伯大神的作品，在我工作中也经常用到，这个模块加载器主要解决了两个问题：一个是命名冲突，模块依赖的问题；另一个是模块的加载和执行。  
基本用法：  

```javascript  
define(function (require, exports, module) {
  var foo = require('foo');
  if (foo.val > 0) {
    var select = require('select1');
  } else {
    var select = require('select2');
  }
  exports.hello = function () {
    return select;
  }
  module.exports.obj = {}
  // ...
});
```
- select的值是根据条件执行的，就是说虽然模块(select1,select2)都下载好了，但是并没有执行factory，预下载，按需执行，更“懒”一些，启动得快了，但也会降低了一点执行的性能。  

**理念**  
根据官网的说法，SeaJS是努力保持简单，专注做浏览器的模块加载。  

### Require.js

**factory执行的时机**  
遵循AMD（异步模块定义）规范，提前加载并执行依赖模块的factory，输出顺序取决于哪个js先加载完或者有指定依赖顺序的以依赖顺序为准。目标是类似于是把执行环境准备好，然后才开始执行。 
基本用法：  

```javascript  
define(['dep1', 'dep2'], function (dep1, dep2) {
  var foo = dep1.val;
  // ...
  return function () {
    return foo;
  };
});
```
如果dep2模块出现了问题，那么在加载的时候就会报错，从而不会执行当前factory函数，但是在seajs中，dep1执行了，而dep2在执行的时候报错了，就可能需要回滚dep1的操作了。  

**理念**  
RequireJS 有一系列插件，功能很强大，但破坏了模块加载器的纯粹性。  

### Webpack
webpack是一个模块打包工具，其可以兼容多种js书写规范（AMD,CMD等），且可以处理模块间的依赖关系，具有更强大的js模块化的功能。官网中用下图描述了webpack采用不同的loader加载不同的资源文件，打包生成多个js文件，也可以根据设置生成独立的图片、css文件等。  
![webpack](/img/in-post/2016-12-06/webpack.jpg)

在Webpack当中，所有的资源都被当作是模块，js，css，图片等等...  
因此，Webpack当中js可以引用css，css中可以嵌入图片dataUrl  

对应各种不同文件类型的资源，Webpack有对应的模块loader  
比如CoffeeScript用的是coffee-loader  

大致的写法是这样子的:  

```javascript  
module.exports = {
  entry:[
    './entry.js',
    // ...
  ],
  output: {
    path: __dirname + '/output/',
    publicPath: "/output/",
    filename: 'result.js'
  },
  module: {
      loaders: [
        { test: /\.css$/, loader: 'style-loader!css-loader' },
      ],
  },
};
```
- 其中entry参数定义了打包后的入口文件，数组中的所有文件会打包生成一个filename文件
- output参数定义了输出文件的位置及名字，其中参数path是指文件的绝对路径，publicPath是指访问路径，filename是指输出的文件名。
- module.loaders 定义了每个模块加载的解析器，所以，被模块化的不止是js了。  

Webpack既是打包工具，又可以做模块加载器，做压缩，并可以加入gulp工作流，非常强大。  

参考：  
[1]. [webpack](http://webpack.github.io/docs/)  
[2]. [Sea.js](http://seajs.org/docs/#docs)  
[3]. [RequireJs](http://requirejs.org/)  


