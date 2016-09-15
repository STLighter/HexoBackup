---
title: Round B APAC Test 2017 题解
date: 2016-09-15 15:09:10
categories: 题解
tags:
  - APAC
  - 算法
  - 题解
---

发现目前博客的流量基本来自APAC的题解, 就来更一发Round B, 依然是D题Large不会的惯例(什么SB惯例).
感觉这套题难度比Round A大, 练习的时候我一直拍挂small(大概我最近状态不好吧).

---

### [A. Sherlock and Parentheses](https://code.google.com/codejam/contest/5254487/dashboard#s=p0)

告诉你`L`和`R`, 你要用`L`个左括号和`R`个右括号, 组成一个字符串. 问组成的字符串最多有多少个子串是"平衡"的(也就是满足括号匹配的).
比如`L = 3`, `R = 2`可以组成字符串`()()(`, 其中子串`1-4`, `1-2`和`3-4`都是"平衡"的, 而且其他组合方式最多也只有`3`个平衡子串, 因此答案是`3`.

其实就全部组成`()`的样式排在一起, 多余的放在后面就是最优的匹配方法, 因为`(())`的形式不会比`()()`更优(前者只有`2`组平衡, 后者有3组). 将一个`()`看作一个位置, 起点终点的取法数就是答案.

```cpp
//省略头文件
typedef long long ll;
void useFile(string f) {
    freopen((f+".in").c_str(),"r",stdin);
    freopen((f+".out").c_str(),"w",stdout);
}
ll l, r;
void gao(){
    scanf("%lld%lld",&l,&r);
    l = l<r?l:r;
    printf("%lld\n", l*(l+1)/2);
}
int main()
{
    useFile("A-large-practice");
    int t;
    scanf("%d",&t);
    for(int i=1;i<=t;++i) {
        printf("Case #%d: ",i);
        gao();
    }
    return 0;
}
```
<!-- more -->

---

### [B. Sherlock and Watson Gym Secrets](https://code.google.com/codejam/contest/5254487/dashboard#s=p1)

给定`A`, `B`, `N`和`K`, 问有多少组`(i, j)`满足$i^A + j^B$能被`K`整除. 其中`i`和`j`为不大于`N`的正整数, 且`i`与`j`不能相等. 