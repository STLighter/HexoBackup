---
title: JavaScript字符串反转
categories: JavaScript
tags:
  - JavaScript
date: 2018-05-04 11:29:39
---

常见面试题(除了一些字符串匹配相关算法会用到似乎并没有什么实际用处?), 看似简单但却有很多谜一样的case.

最常见的解法就是将字符串切成数组, 再利用数组的`reverse`方法反转后再拼接起来, 如下:

```
function reverse(str) {
  return Array.prototype.slice.call(str).reverse().join('');
}

reverse('abcde'); // "edcba"

reverse('我系渣渣辉'); // "辉渣渣系我"
```

这样的做法在遇到四字节字符时会出现问题, 因为在`slice`的时候一个四字节字符会被分成两个数组元素.

例如里面包含一个笑脸的emoji.
```
reverse('咕嘿嘿😃'); // "��嘿嘿咕"
```

甚至有些看起来会比较诡异.

```
reverse('是慌?还是慌?还是慌?'); // "?��是还?��是还?慌是"
```

上面三个"慌"字看上去好像是一样的, 但后面两个"慌"是四字节字符, 甚至这两个"慌"的字符编码也不一样.

那么这又要怎么解决呢?

<!-- more -->

`ES6`对字符串添加了迭代器, 并且这个迭代器可以识别四字节字符. 可以用`...`将字符串切成数组, 这样可以保证四字节字符不会被分成两个.

```
function reverse(str) {
  return [...str].reverse().join('');
}

reverse('咕嘿嘿😃'); // "😃嘿嘿咕"

reverse('是慌?还是慌?还是慌?'); // "?慌是还?慌是还?慌是"
```

也可以利用正则表达式将四字节字符的前两个字节与后两个字节先交换一次, 再用`slice`的方式进行反转.

```
function reverse(str) {
  str = str.replace(/([\uD800-\uDBFF])([\uDC00-\uDFFF])/g, (_, $1, $2) => `${$2}${$1}`);
  return Array.prototype.slice.call(str).reverse().join('');
}

reverse('是慌?还是慌?还是慌?'); // "?慌是还?慌是还?慌是"

reverse('咕嘿嘿😃'); // "😃嘿嘿咕"
```

这里正则的依据是`UTF16`编码规则.

这样的解法看似是最终答案, 直到遇到这个case(来源于[esrever](https://github.com/mathiasbynens/esrever)).

```
reverse('mañana mañana'); // "anãnam anañam"
```

注意`~`的位置.

原因在于其中包含一种特殊的字符, 其作用是装饰前一个字符(装饰字符甚至可以叠加多个). 例如字符串中第二个`~`. 这个字符的编码为`\u0303`.

```
'\u0303' \\ "̃"

'a\u0303' \\ "ã"

'a\u0303\u0303' \\ "ã̃"

'a\u0303\u0305' \\ "ã̅"
```

解决办法可以看[esrever的实现](https://github.com/mathiasbynens/esrever/blob/master/esrever.js#L20).

其创建了两个正则表达式, 第一个`regexSymbolWithCombiningMarks`用来匹配普通字符及其附带的装饰字符, 第二个`regexSurrogatePair`用来匹配四字节字符.

反转时先匹配`regexSymbolWithCombiningMarks`, 将附带的装饰字符部分反转后放在普通字符前面; 接着再匹配所有四字节字符, 并交换前后两字节位置; 最后再执行普通的反转操作(与`slice`方法效果相同).

`regexSymbolWithCombiningMarks`是由`allExceptCombiningMarks`和`combiningMarks`拼接而成, 而这两者的计算在[/scripts/export-data.js](https://github.com/mathiasbynens/esrever/blob/master/scripts/export-data.js)中, 可以看到装饰字符是从一个预先创建好的字符分类表得到的, 通过`regenerate`计算出`allExceptCombiningMarks`和`combiningMarks`的正则, 再由Grunt替换源码中的变量得到`regexSymbolWithCombiningMarks`.

所以...如果真有这种特殊需求的话, 还是老老实实用`esrever`吧.

### 参考文献

[ECMAScript 6 入门 - 扩展运算符的应用](http://es6.ruanyifeng.com/#docs/array#%E6%89%A9%E5%B1%95%E8%BF%90%E7%AE%97%E7%AC%A6%E7%9A%84%E5%BA%94%E7%94%A8)

[esrever](https://github.com/mathiasbynens/esrever)

[维基百科UTF-16](https://zh.wikipedia.org/wiki/UTF-16)

[字符码点查询](https://codepoints.net/)
