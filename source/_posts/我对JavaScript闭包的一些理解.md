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

实际上, 目前JavaScript引擎还不会去检测一个函数是否真的会被使用到, 因此只要创建函数, 这个函数就会"记住"自己创建时的环境, 无论这个函数是否会返回或者被用到. 比如以下代码执行也会产生闭包.

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
    // 可执行代码, 在debugger处查看compareNames的<function scope>
    debugger;
}
createComparisonFunction("name");
```

代码中`compareNames`函数既没有被返回, 也没有被使用, 但在Chrome下, 我们都能观察到`compareNames`的`<function scope>`中包含有`propertyName`, 也就"记住"了其创建时的环境, 成为了闭包. 这个闭包与第一份代码的闭包没有什么区别, 只是在内存回收上有些不同(这个后面也会说到).

### 闭包如何"记住"创建时的环境

从上面的描述中可以知道, 闭包在函数创建时就产生了, 那么他是通过什么方式来"记住"当前环境的呢? 答案就是`作用域链`.
在函数对象创建时, 会在`[[Scope]]`属性中保存当前执行环境的作用域链, 全局执行环境的作用域链只有全局变量对象(执行环境和变量对象是不同的东西, 变量都是放在变量对象中的).
当函数在执行时, 会创建一个新的执行环境, 执行环境的作用域链将会指向一个作用域链对象. 这个作用域链对象从函数的`[[Scope]]`拷贝而来, 并且会在其前端加入一个新的活动对象, 用于存放参数等变量(也就是当前函数执行的作用域).
我们看下面的代码:

```JavaScript
function foo() {
    // 2
    debugger;
}
// 1
debugger;
foo();
```

![debugger-1](/uploads/scope1.png)

在`1`处的`debugger`我们能够观察到当前执行环境(`Window`)的作用域链(图中`Scope`部分)中只有`Global: Window`. 同时, `foo`的`<function scope>`与当前执行环境的作用域链相同.

![debugger-2](/uploads/scope2.png)

当继续执行到`2`处, 这时当前执行环境为`foo`, 可以看到`Scope`下多了`Local`, 这里的`Local`就是`foo`的活动对象.

// TODO: Draw Scope

有了作用域链对象, 函数执行时便可以从前往后访问作用域之外的变量了.
再看一个复杂一点的例子:

```JavaScript
function foo() {
    var name = "ST_Lighter";
    function bar(prefix) {
        return prefix + name;
    }
    return bar;
}
var o = foo();
o("My name is ");
```

动手画一画作用域链吧.

// TODO: Draw Scope

在IE中通过`debugger`观察闭包, 可以看到完整的作用域链, 以及链上对象所包含的变量. 然而在Chrome中, 结果却不完全相同.
使用下面的代码进行调试:

```JavaScript
function foo() {
    var name = "ST_Lighter";
    var blog = "http://stlighter.github.io";
    function bar(prefix) {
        debugger;
        return prefix + name;
    }
    return bar;
}
var o = foo();
o("My name is ");
```

这是我在IE10中的结果, 可以看到完整的作用域链, 链上包含了`foo`的活动对象.

![Scope in IE](/uploads/IEScope.png)

而在Chrome中, 作用域链上并没有完整的`foo`活动对象, 没有被使用过的变量`blog`不见了.

![Scope in Chrome](/uploads/ChromeScope.png)

这是因为V8对闭包进行了优化. V8并不是将整个活动对象装进作用域链, 而是在函数执行时创建一个`Context`, 会被下面创建的函数所使用的变量将被放入`Context`(也还有其他方法让变量进入`Context`). 当创建函数时, 作用域链前端不是当前执行函数的活动对象, 而是`Context`. 更详细内容请参考[Grokking V8 closures for fun (and profit?)](http://mrale.ph/blog/2012/09/23/grokking-v8-closures-for-fun.html).
V8这样做是为了更有效的回收无用的对象, 释放内存空间, 这点在后面内存回收部分细讲.

(未完待续..)

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