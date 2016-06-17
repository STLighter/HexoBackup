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