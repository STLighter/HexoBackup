---
title: CSS多行文字水平垂直居中的几种方式
date: 2016-09-12 11:42:35
categories: CSS
tags:
  - CSS
---

单行文本的水平垂直居中常通过设置`line-height`实现, 而当我们需要使多行文字水平垂直居中时, 直接设置`line-height`不能够实现我们所需的效果.
下面通过四种不同思路实现多行文字的水平垂直居中. 这里主要介绍思路, 兼容性没有详细测过, 对于不同浏览器可能需要微调(我是在chrome 51下面跑的).

HTML如下:
```HTML
<div id = "test">
    <p>
        MY TEST
        MY TEST
    </p>
</div>
```
<!-- more -->

---

### 使用inline-block, 通过设置文本对齐实现居中

基本思想是用子容器包裹文字, 并将子容器在父容器中水平垂直居中. 这里通过设置文本对齐实现子容器居中.

要让子容器包裹文字且能受文本对齐属性的控制, 需要将子容器设为`inline-block`. 要使得子容器水平垂直居中, 需要在父容器上设置`text-align: center`; 在子容器上设置`vertical-align: middle`并且将父容器的`line-height`设置为父容器的高度.

值得注意的是, `line-height`是会继承的, 所以子容器需要将`line-height`重置, 使得其内部的文本能够正常显示.

要能够看到效果, 还需要在HTML中父容器内添加文本, 这里通过`::after`来做.

为了使得子容器完全水平垂直居中, 同时隐藏父容器中多余的文本, 设置`font-size: 0`; 同时, 由于继承的关系, 子容器要重新设置`font-size`.(似乎设置过小的字体在webkit上可能会出现bug?)

```CSS
#test {
    line-height: 100px;
    background-color: black;
    text-align: center;
    font-size: 0;
}
#test p {
    display: inline-block;
    width: 70px;
    line-height: normal;
    vertical-align: middle;
    font-size: 16px;
    color: white;
}
#test::after {
    content: "$";
}
```

### 使用block, 通过设置子容器位置实现居中

基本思想与思路1类似, 只是这里是通过设置位置属性实现块元素的水平垂直居中.

首先要让子容器为`block`. 这里`<p>`本来就是块级元素, 无需设置.

设置父容器`position: relative`; 子容器`position: absolute`. 并设置子容器`left: 50%`和`top: 50%`使得子容器左上角在父容器中央.

通过`transform: translate(-50%, -50%)`使得子容器中间而不是左上角定位到父容器中央. 并设置`text-align: center`使得子容器内部文本也居中.

```CSS
#test {
    position: relative;
    height: 100px;
    background-color: black;
}
#test p {
    width: 70px;
    position: absolute;
    text-align: center;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    color: white;
}
```

### 使用table布局

父容器设置`display: table`, 子容器设置`display: table-cell`, 并通过`vertical-align: middle`和`text-align: center`实现文字水平居中, 容器垂直居中.

值得注意的是, 这种方法无法设置子容器的宽度, 要让文本换行需要加`<br\>`或者设置`padding`. 

```CSS
#test {
    display: table;
    width: 100%;
    height: 100px;
    background-color: black;
}
#test p {
    display: table-cell;
    text-align: center;
    vertical-align: middle;
    color: white;
}
```

### 使用flex布局

兼容性稍差, 对于IE10等浏览器要添加prefix甚至属性名称/值都有不同.

父容器设置`display: flex`, 并用`justify-content: center`实现子容器水平居中, `align-items: center`实现子容器垂直居中.

子容器设置`text-align: center`水平居中文本.

```CSS
#test {
    height: 100px;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: black;
}
#test p {
    width: 70px;
    text-align: center;
    color: white;
}
```