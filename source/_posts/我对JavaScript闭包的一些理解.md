---
title: 我对JavaScript闭包的一些理解
date: 2016-09-13 12:14:35
categories: JavaScript
tags:
  - JavaScript
  - Closure
---

面试老被问到闭包相关的问题, 之前也一直答的不是很好. 最近研究了一下闭包相关的一些内容, 以下是我个人的一些理解.

---

### 闭包指什么

> Closures are functions that refer to independent (free) variables (variables that are used locally, but defined in an enclosing scope). In other words, these functions 'remember' the environment in which they were created.

这段话来自[MDN - Closures(Last updated by: SphinxKnight, Sep 8, 2016, 1:06:52 AM)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures), 简单的说就是: `当一个函数中使用了其他作用域中的变量, 它就是一个闭包. 闭包能够"记住"其创建时所在的环境`.

下面还有一段解释:

> A closure is a special kind of object that combines two things: a function, and the environment in which that function was created. The environment consists of any local variables that were in-scope at the time that the closure was created.`

这里说`闭包是含有一个函数以及创建时所处环境的对象`.

可以看出, MDN上对闭包的指代并不明确, 并不知道是指代函数还是函数加创建环境所组成的对象.

《JavaScript高级程序设计》中对闭包的解释是:

> 闭包是指有权访问另一个函数作用域中的变量的函数

其后对闭包的详细描述也都是将函数视为闭包的.
结合网络上其他的一些解释, 我认为:`闭包指的是一个函数`.
当然, 这个函数需要能"记住"创建时所在环境, 具体细节后面会讨论.
<!-- more -->

### 如何产生闭包

这里使用《JavaScript高级程序设计》中的例子.

```JavaScript
function createComparisonFunction(propertyName) {
    return function(object1, object2) {
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];

        if(value1 < value2) {
            return -1;
        } else if(value1 > value2) {
            return 1;
        } else {
            return 0;
        }
    }
}

var compareNames = createComparisonFunction("name");
var result = compareNames({name: "Nicholas"}, {name: "Greg"});
```

这段代码中, `createComparisonFunction`内的匿名函数能够访问执行`createComparisonFunction`时`createComparisonFunction`内的变量`propertyName`, 也就满足了`访问其他作用域中变量`这个条件, 因此这个匿名函数就是一个闭包.

大多讲解闭包的资料上都使用的类似的代码, 有些资料上在解释的时候认为`返回内部的匿名函数`也是一个特点, 造成`需要返回内部函数才叫闭包`的误解.

实际上, 目前JavaScript引擎还不会去检测一个函数是否真的会被使用到, 因此只要创建函数, 这个函数就会记住自己创建时的环境, 无论这个函数是否会返回或者被用到. 比如以下代码执行也会产生闭包.

```JavaScript
function createComparisonFunction(propertyName) {
    // 这个函数也是闭包, 即使没有被使用过
    function compareNames(object1, object2) {
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];

        if(value1 < value2) {
            return -1;
        } else if(value1 > value2) {
            return 1;
        } else {
            return 0;
        }
    }
    // 可执行代码, 在debugger处查看compareNames的作用域查看其绑定的环境
    debugger;
}
createComparisonFunction("name");
```

代码中`compareNames`函数既没有被返回, 也没有被使用, 但是无论在IE还是Chrome下, 我们都能观察到`compareNames`的作用域链(这个后面再细说)中包含有`propertyName`, 也就"记住"了其创建时的环境, 成为了闭包. 这个闭包与第一份代码的闭包没有什么区别, 只是在内存回收上有些不同(这个后面也会说到).

(未完待续.. 应该会在近几天内更完)

### 闭包如何"记住"创建环境
(高程上的作用域链详解与Chrome对闭包的优化)
### 再看闭包定义
(不同浏览器下闭包不完全相同)
### 闭包与内存回收
(不同浏览器下闭包造成的内存泄漏)
### 无时不刻都在创建闭包?
(全局作用域下创建放函数算不算闭包?)
### 细节上的咬文嚼字
(函数作用域? consists of any local variables that were in-scope AT THE TIME that the closure was created?)

---

参考资料:
[MDN Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
[JavaScript高级程序设计（第3版）](https://book.douban.com/subject/10546125/)
[Grokking V8 closures for fun (and profit?)](http://mrale.ph/blog/2012/09/23/grokking-v8-closures-for-fun.html)
[A surprising JavaScript memory leak found at Meteor](http://point.davidglasser.net/2013/06/27/surprising-javascript-memory-leak.html)