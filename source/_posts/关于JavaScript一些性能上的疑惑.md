---
title: 关于JavaScript一些性能上的疑惑
date: 2016-06-08 19:55:33
categories: JavaScript
tags:
  - JavaScript
---

最近作死的用JavaScript去试着做Codeforces上的题,在一道数据范围比较极限的题上栽了跟头([Codeforces Round #354 (Div. 2) - D. Theseus and labyrinth](http://codeforces.com/contest/676/problem/D)).一道简单的广搜题,结果不是tle就是mle.最后各种调整,终于算是搞过去了,也给我留下了一些关于JS语言上的疑问,希望以后能有时间看看JS引擎,解决这些谜题.

---

### 谜题1. Array的shift()/unshift()的等操作复杂度到底是多少?

广搜一般的实现方法就是使用一个队列存放接下来要搜的点,然而我使用JS中的队列方法shift()居然tle了.由于实在不了解shift()具体实现是怎样的,抱着试一试的心态自己实现了一个队列才解决了tle的问题.难道shift()要移动后面所有的元素,使得其复杂度是O(n)?如果用类似C++的容器实现方式应该会很高效啊.
<!-- more -->

### 谜题2. 如果Array访问(赋值和取值)的位置超过其当前长度,会对其性能造成什么影响?

另一个造成tle的地方也使我很迷惑.我使用了一个数组用来记录某个点是否访问过,以便剪枝.对于JS来说,不在严格模式下,对越界的空间赋值和访问都是没问题的,数组会自动扩容.利用这个特性,声明了数组直接赋值就行了.然而不给length赋值tle了,手动赋值length却AC了.不知道具体实现是否像C++中一样有"剩余容量"这种设计,有的话不至于这么低效吧.

---

### 后记

查看了V8的源码,单从ArrayShift和ArrayUnshift代码来看每次操作应该是$O(n)$的复杂度.但在实际测试下,当数组大小超过某一个值时,shift耗时暴增,应该底层有其他优化,使得在数据较少时能达到近似$O(1)$的复杂度,需要进一步研究.测试代码如下.

```javascript
var N = 223781;
var a_pop = [];
var a_shift = [];
function main() {
    for(var i = 0; i < N; i++) {
        a_pop.push(i);
    }
    for(var i = 0; i < N; i++) {
        a_shift.push(i);
    }

    start = Date.now();
    for(var i = 0; i < N; i++) {
        a_pop.pop();
    }

    print("Pop Done in " + (Date.now() - start));

    start = Date.now();
    for(var i = 0; i < N; i++) {
        a_shift.shift();
    }
    print("Shift Done in " + (Date.now() - start));
}

main();
```

问题2我只做了简单的测试,现象很复杂,而且直接在V8上跑和在浏览器里跑效果还不太一样(估计是版本的问题).不过可以肯定的是,如果数据较多,且不是有序赋值的话,先定义Array的长度效率会比较高.比如下面代码的访问顺序,不先定义数组总长度就会很慢.

```javascript
var Step = 1025;
var Loop = 1024;
var N = Step * Loop;
var a_setL = [];
var a_unsetL = [];
function main() {
    var start = Date.now();
    a_setL.length = N;
    for(var i = Step - 1, tmp = 0; i < N; tmp += Step, i += Step) {
        for (var j = i; j >= tmp; j--) {
            a_setL[j] = j;
        }
    }

    print("Set Length Done in " + (Date.now() - start));

    start = Date.now();
    for(var i = Step - 1, tmp = 0; i < N; tmp += Step, i += Step) {
        for (var j = i; j >= tmp; j--) {
            a_unsetL[j] = j;
        }
    }
    print("Didn't Set Length Done in " + (Date.now() - start));
}

main();
```

具体的原理应该和Array随机访问的方式有关,这一点需要进一步研究.

---

### 后记2

拜读了Jay Conrod的[A tour of V8: object representation](http://www.jayconrod.com/posts/52/a-tour-of-v8-object-representation)(中文版戳[这里](http://newhtml.net/v8-object-representation/)).
根据里面的描述,数组通常使用Fast Element, 但如果在远远超过当前数组大小的地方赋值的话,数组会被转成字典模式(访问较慢).这意味着如果需要一个固定大小的数组,而且需要随机访问的话,相比不事先指定长度,或者给数组赋值length,按顺序加length个初始值进去再访问效率更高.因为这样不会使得数组的属性访问变成字典模式(不过如果访问是稀疏的,赋值length个值可能消耗更多,这个看实际情况取舍了)

```javascript
var Step = 1025;
var Loop = 1024;
var N = Step * Loop;
var a_setL = [];
var a_unsetL = [];
function main() {
    var start = Date.now();
    for(var i = 0; i < N; ++i) {
        a_setL[i] = 0;
    }
    for(var i = N - 1; i >= 0; --i) {
        a_setL[i] = i;
    }
    
    print("Set Length Done in " + (Date.now() - start));

    start = Date.now();
    for(var i = N - 1; i >= 0; --i) {
        a_unsetL[i] = i;
    }
    print("Didn't Set Length Done in " + (Date.now() - start));
}

main();
```

嘛,等我再消化消化而且另一个问题有了眉目之后,大概会删掉这篇,另外写一篇总结吧~

---

### 后记3

卧槽,shift的坑居然在heap的实现里面.在dalao的指引下看了[这个](http://stackoverflow.com/questions/27341352/why-does-a-a-nodejs-array-shift-push-loop-run-1000x-slower-above-array-length-87).根据Andras的回答,数组的大小不同决定了它在堆里存放的位置.小的数组(我猜是放在年轻分代里)在执行移动元素的操作时,其实在堆中只是移动了指针而已.当大小超过一定数值,数组将会被放到一个用于存放大对象的大对象空间(一页一个对象),而由于内存对齐的原因(大概是页对齐?)不能通过移动指针实现,只能真实的在内存中移动数据,因此效率降低.