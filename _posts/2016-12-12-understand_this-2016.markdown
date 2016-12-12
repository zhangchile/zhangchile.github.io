---
layout: post
title : 理解js中的this
subtitle: ""
date: 2016-12-12
author: "chile.zhang"
header-img: "img/post-bg-2016.jpg"
tags:
    - Javascript
    - 前端
    - this
---

## 理解js中的this
以前写代码，对this的了解还是比较简单，比如在window下面this都是window，在一个function里this就是这个function，真的不清楚就直接console.log(this)看看，这些都是从表面现象得出的结论。但是后来理解了JavaScript函数执行方式，对this就永远不会困惑了。  
先来看一段代码：  
{% highlight javascript %}
var obj = {
  foo: function () {
    console.log("obj.foo() called");
    console.log(this);// foo
  }
};
obj.foo(); // object foo

var other = obj.foo;
other(); // window

obj.foo.call({});// object {}

obj.foo.call();// window
{% endhighlight %}
事实上，**JavaScript在执行函数的时候，都是用function.call(context, arg1,arg2,...)来执行的。**  
而当我们调用obj.foo()的时候，那么执行的就是obj.foo.call(obj)，所以打印的就是obj这个对象；同理当我们调用obj.foo.call({})，那么上下文就是{}，所以this就是{}。  
而当我们调用other()的时候，那么执行的就是other.call();**context为空时会默认填window对象**，所以打印的就是window对象。  

用上面的方法，再来看一个例子：  
{% highlight javascript linenos %}
var a = 123;
function fn() {
    console.log( this.a );
}
var obj2 = {
    a: 42,
    fn: fn
};
var obj1 = {
    a: 2,
    obj2: obj2
};
obj1.obj2.fn(); // 42 -- this引用的是obj2
{% endhighlight %}
那么实际的调用方式是obj1.obj2.fn.call(obj2)，this指向的是直接上一级的对象，如果要让this引用的是window对象，只需obj1.obj2.fn.call();这样输出的就是全局的a的值123了。  
所以，只要弄明白call的上下文就不搞错this的值了。


