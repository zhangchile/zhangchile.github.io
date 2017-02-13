---
layout: post
title : Angular的$digest循环理解
subtitle: ""
date: 2017-02-13
author: "chile.zhang"
header-img: "img/post-bg-2016.jpg"
tags:
    - Angular
    - Javascript
    - 前端
---

## Angular的$digest循环理解
angualr的双向数据绑定，底层靠的是个称为“digest loop”的机制。这个digest循环主要有两部分实现：  
- $watch list  
- $evalAsync list  

### $watch list
angular中，每次我们在页面触发一个事件（如:修改input内容的change）， 都会触发一个回调函数，去更新对应的model。  
例子：  

```html   
<!DOCTYPE html> 
<html ng-app> 
<head> 
  <title>Simple app</title> 
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.js"> 
  </script> 
</head> 
<body> 
  <input ng-model="name" type="text" placeholder="Your name"> 
  <h1>Hello {{ name }}</h1> 
</body> 
</html>
```

上面的例子，我们修改input里的值，就会立刻在下面的h1中显示出来。这是因为通过ng-model把input的值绑定到了$scope.name属性，angular为了能做到双向数据绑定，就会增加一个监听函数到`$watch list`，那么一但name的值变了，angular就会更新对应的html了。值得注意的是，所有的ui元素，如果绑定到了$scope，那么angualr就会自动增加一个$watch函数到`$watch list`中，当执行这个$digest循环的时候，有个dirty check的过程，这时$watch函数将会被解释执行了。

### dirty checking
dirty checking 处理的方式其实很简单，就是检查一个值在被应用同步之前是不是被修改了。  
angular中会保持观察watches（在$watch对象中）中的值，顺序遍历`$watch list`，如果要更新的值和旧的值没有变化，就会继续往下遍历。如果这个值发生了改变，angular就记录这个新的值，然后继续遍历。一但遍历完了`$watch list`，如果有任何值发生了改变，angular会再次遍历整个list，直到没有值任何变化。为什么要重新遍历整个list，因为如果我们更新一个值可能会导致另一个值更新，这样的话angular就不会检查到另外的值更新了。  
但是如果循环超过了10次或更多，angular会抛出异常，并终止掉。否则应用会导致死循环。
![webpack](/img/in-post/2017-02-13/digestLoop.png)

### $watch 函数
在$scope对象上的$watch函数，会在angular的$disgest循环中设置dirty check，如果$watch的表达式发生了变化，angualr就会返回这个对象。  
$watch函数包含三个参数：  
- watchExpression
watchExpression可以是一个属性或者一个函数，将会在每个$digest循环中执行。  
如果watchExpression是一个字符串，angualr就会把解释为$scope里的属性，如果是一个函数，就会根据函数返回的值进行判断。
- listener/callback
回调函数是当watchExpression发生改变（新旧值不一样）的时候，调用的函数
- objectEquality (optional)
boolean参数，告诉angualr是否按照严格对等（===）来检查

$watch函数会返回一个注销函数，用来告诉angular取消wath这个表达式：  

```javascript   
// ... 
var unregisterWatch = $scope.$watch('newUser.email',   function(newVal, oldVal) {
  if (newVal === oldVal) 
    return; // on init 
}); 
// ... 
// later, we can unregister this watcher 
// by calling 
unregisterWatch();
```

再举个例子，如果我们要获得一个输入框里的姓和名，  

```html   
<input type="text" ng-model="full_name" placeholder="Enter your full name" />
```

注意，将$watch写到controller不是最佳实践，这样会很难测试controller，最好是放到services里。  
我们可以给full_name添加一个$watcher来实现，当输入一个全名的时候，自动识别姓和名。  

```javascript   
angular.module("myApp").controller("MyController", ['$scope', function ($scope) {
    $scope.$watch('full_name', function (newVal, oldVal, scope) {
        // the newVal of the full_name will be available here 
        // while the oldVal is the old value of full_name 
        if (newVal === oldVal) {
          // This will run only on the initialization of the watcher
        } else {
          // A change has occurred after initialization
          // do somthing to split first name and last name
        }
    });
}]);
```

在应用初始化的时候，会调用一次回到函数，所以第一次的时候，newVal和oldVal将会是undefined的，同时也是相等的，所以如果我们要初始化值，可以简单的通过newVal === oldVal判断就好。