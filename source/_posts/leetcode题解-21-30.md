---
title: leetcode题解(21-30)
date: 2016-12-23 21:18:16
categories: 题解
tags:
  - leetcode
  - 算法
  - 题解
---

摸鱼多日, 来随便更一更. 最近JS写的比较多, 所以以后题解的代码就以JS为主了. 这篇题解写了好久, 主要是后面两题搞得有点复杂, 加上各种学校的事情, 拖着写了个把月QAQ.

---

### [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)

合并两个有序链表, 链表中数据是从小到大排序的.
这有点像归并排序中归并的过程, 只是这个要在链表上进行归并, 思路还是一样的.
对两个链表头中的值相互比较, 取值较小的那个作为新链的下一个节点, 并将取出的链表头向后移动一位, 并继续这一个过程直到两个链表都被遍历.
复杂度`O(N+M)`, `N`和`M`为两个链表长度.
代码中为了方便我建了一个空节点作为头节点, 并返回了一个新链表, 而没有破坏原先两个链表. 只需要稍微修改一下就能直接使用原先链表的节点.

```JavaScript
var mergeTwoLists = function(l1, l2) {
    let head = new ListNode(0); // 空的头节点
    let tmp = head; // 标记链表尾, 方便直接在尾添加
    // 循环直到两个链表都清空
    while(l1||l2) {
        let target; //新链表的下一个节点值
        // 如果两个节点都存在, 获取其中值小的一个
        if(l1&&l2) {
            if(l1.val<l2.val) {
                target = l1.val;
                l1 = l1.next;   // 下一个取l1的头, 下一次将比较l1的下一个节点与l2的头的大小
            } else {
                target = l2.val;
                l2 = l2.next;
            }
        } else {
            // 两个链表中有一个已经遍历完, 其值为null
            if(l1){  
                target = l1.val;
                l1 = l1.next;
            } else {
                target = l2.val;
                l2 = l2.next;
            }
        }
        // 将新值加入新链表尾部
        tmp.next = new ListNode(target);
        tmp = tmp.next;
    }
    return head.next;
};
```
<!-- more -->

---

### [22. Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)

给`N`对小括号, 返回所有合法的括号序列.
典型的分治问题. 要保证一个括号序列合法, 其每个括号内的子序列也要是合法. 我们可以根据第一对匹配括号的大小(其中含有的括号数)来枚举.
如`N = 1`有唯一的合法序列`()`.
如`N = 2`, 则有序列`()()`和`(())`. 等价于1对括号加所有`N = 1`的序列或者1对括号内含`N = 1`的序列.
如`N = 3`, 我们在1对括号后加上所有`N = 2`的序列可以得到`()()()`和`()(())`;
然后我们在1对括号里放所有`N = 1`的序列, 并在其后加上所有`N = 1`的序列, 得到`(())()`;
我们在1对括号里放所有`N = 2`的序列, 则可以得到`(()())`和`((()))`.

即要得到`N`组括号的序列, 我们可以让`i`组括号的序列在1对括号内, 再在后面加`N - i - 1`组括号序列, 于是得到规模为`i`和`N - i - 1`的子问题.

我们用数组记录一下子问题的答案便可以从小到大递推得到答案了. 至于复杂度, 由于答案规模是指数级增加的, 所以复杂度也是指数级的, 比较爆炸.

```JavaScript
var generateParenthesis = function(n) {
    let dp = new Array(n + 1);  // 数组记录子问题的答案 
    dp[0] = [''];   // 规模为0的子问题答案
    // 递推1到n每个问题
    for(let i = 1; i <= n; ++i) {
        dp[i] = [];
        // j为第一对括号包含的括号对数(包括自己)
        for(let j = 1; j <= i; ++j) {
            // k为需要在后面加上的括号对数
            let k = i - j;
            // 枚举两个子问题答案的所有组合
            for(a of dp[j - 1]) {
                for(b of dp[k]) {
                    dp[i].push('(' + a + ')' + b);
                }
            }
        }
    }
    return dp[n];
};
```

---

### [23. Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

合并`k`个有序链表, 链表中数据是从小到大排序的.
和21题如出一辙, 就是多路归并的思路. 从所有的头中取最小的一个作为新链表的头. 为了提高选取效率, 可以使用优先队列去实现. JS里面没有优先队列, 所以我手敲了一个, 具体的算法可以去搜索`大/小顶推`, 网上有许多资料, 《算法导论》中关于堆排序的部分也有讲解.
整体复杂度`O(Nlog(k))`, 其中`N`为链表中点的总个数, `k`为链表数.

```JavaScript
var mergeKLists = function(lists) {
    // 优先队列
    class PriorityQueue {
        constructor(cmp) {
            this.arr = [];  // 存放实际节点的容器
            this.cmp = cmp; // 比较函数, 用于比较节点的大小
        }
        get size() {
            return this.arr.length; // 队列大小
        }
        empty() {
            return this.size === 0; // 队列是否为空
        }
        top() {
            if(this.size > 0)
                return this.arr[0]; // 返回队首(堆顶)
        }
        swap(i, j) {
            // 交换数组中元素i, j的位置
            let arr = this.arr;
            let tmp = arr[i];
            arr[i] = arr[j];
            arr[j] = tmp;
            return this;
        }
        push(o) {
            // 插入新节点
            let arr = this.arr, 
                i = arr.length, // 新节点角标
                p;
            arr.push(o);    // 加在数组尾部(保持完全二叉树结构)
            while(i > 0) {
                // 维护堆性质, 向上交换直到父节点大于当前节点或当前节点为根
                p = parseInt((i - 1)/2);    // 父节点角标
                if(this.cmp(arr[p], arr[i]) >= 0)   // 当父节点较大则不再交换
                    break;
                this.swap(p, i);    // 父节点较小, 与当前节点交换
                i = p;  // 更新当前节点的实际角标
            }
            return this;
        }
        pop() {
            // 删除队首节点
            if(this.size === 0)
                return;
            let arr = this.arr,
                cmp = this.cmp,
                l = arr.length - 1, // 队尾角标
                ret = arr[0],   // 要删除的节点
                i = 0,  // 队首角标
                p = 1,  // 队首左二子角标
                q = 2,  // 队首右儿子角标
                t;
            arr[0] = arr[l];
            arr.pop();  // 等价于队首队尾交换并移除原队首, 新队首可能不满足性质
            while(p < l) {
                // 维护大顶堆性质, 从根开始, 将其与左右儿子中较大的交换并继续向下比较, 直到左右儿子都比其小
                if(cmp(arr[i], arr[p]) >= 0) {
                    t = i;
                } else {
                    t = p;
                }
                if(q < l && cmp(arr[q], arr[t]) > 0) {
                    t = q;
                }
                if(i === t) // 当父节点为最大的节点
                    break;
                this.swap(i, t);    // 将当前节点与较大的儿子交换
                i = t;  // 更新当前节点角标
                p = i*2 + 1;    // 更新左二子角标
                q = p + 1;  // 更新右儿子角标
            }
            return ret;
        }
    }
    let queue = new PriorityQueue((a, b) => {
        return b.val - a.val;
    });
    lists.forEach((list) => {
        if(list) {
            queue.push(list);
        }
    });
    let head = new ListNode(-1);
    let tmp = head;
    while(!queue.empty()) {
        tmp.next = queue.pop();
        tmp = tmp.next;
        if(tmp.next) {
            queue.push(tmp.next);
        }
    }
    return head.next;
};
```

---

### [24. Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/)

将链表中的元素两两交换, 如`1->2->3->4->5`, 则交换`1`与`2`和`3`与`4`, 多出的`5`不管, 返回`2->1->4->3->5`.
简单的递归. 要两两交换所有元素, 等价于先交换除去前两个节点之外的其他节点, 然后再交换前两个节点. 交换其他节点便形成子问题.
由于交换后头节点会变化, 最方便的方法就是将新的头结点作为返回值传递.
复杂度`O(N)`.

```JavaScript
var swapPairs = function(head) {
    // 递归终止条件, 如果没有节点或者只有一个节点, 则直接返回
    if(!head) return head;  // 已经到链表末尾, 返回空
    let tmp = head.next;    // tmp为当前节点的相邻节点, 即需要与当前节点交换
    if(!tmp) return head;   // 单节点, 没有相邻节点, 返回原节点
    head.next = swapPairs(tmp.next);    //  递归得到当前节点和其相邻节点之后的链表交换结果, 链在当前节点之后
    tmp.next = head;    // tmp的下一个点设为当前节点, 实现交换, tmp为新的头结点
    return tmp; // 返回新的头结点
};
```

---

### [25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

上一题的升级版, 给一个链表和一个长度`k`, 将每连续长度为`k`链翻转, 尾部长度不足`k`的部分保持不变.
可以先用递归实现一个翻转链表前`k`个节点的函数.
对于`1->2->3->4->5`, `k = 3`, 我们可以先找到`3`, 并将其前导放到`3`后得到`1->3->2->4->5`, 然后再将`3`新的前导`1`放到`2`后得到`3->2->1->4->5`.
有了这个函数, 可以先从根节点做一次翻转, 如果翻转完成, 则旧的根节点已经成为第`k`个节点, 然后继续从其后一个节点开始再做翻转, 直到无需翻转即可.
复杂度`O(N)`.

```JavaScript
var reverseKGroup = function(head, k) {
    function reverseAll(bf, cur, ct) {
        // ct为计数值, 当ct为0时表示这个点是需要翻转的链的尾部尾
        if(!cur) return null;   // 未满足计数就达到链尾, 说明无需翻转
        let ret;
        if(!ct) {
            // 需要翻转的链的尾部, 其作为新的头节点
            ret = cur;
        } else {
            // 从后往前翻转
            ret = reverseAll(cur, cur.next, ct - 1);
        }
        if(ret && bf) {
            // 需要翻转, 将前导节点放到当前节点后面
            bf.next = cur.next;
            cur.next = bf;
        }
        return ret;
    }
    let ret = new ListNode(),
        tmp = ret;
        ret.next = head;
    while(head) {
        // 每k个依此翻转
        tmp.next = reverseAll(null, head, k - 1);
        if(!tmp.next) {
            tmp.next = head;
            break;
        }
        tmp = head;
        head = head.next;
    }
    return ret.next;
};
```

---

### [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

去除有序数组中的重复项, 返回新数组长度, 要求只能使用`O(1)`的额外空间.
因为数组有序, 所以相同项一定相邻. 要删除重复项, 只需要判断每一项是否与前一项相同, 相同的话删除即可. 这样对于要保留的项, 我们直接将其移动到合适的位置即可. 因为其一定是往前移动, 所以只需要从前往后枚举, 然后覆盖到原先的数组上即可.

```JavaScript
var removeDuplicates = function(nums) {
    let l = nums.length,
        p = 1;
    for(let i = 1; i < l ; ++i) {
        if(nums[i] === nums[i-1])
            continue;
        nums[p++] = nums[i];
    }
    return p;
};
```

---

### [27. Remove Element](https://leetcode.com/problems/remove-element/)

去除有序数组中的指定值的项, 返回新数组长度, 要求只能使用`O(1)`的额外空间.
与上一题类似, 跳过要删除的项, 对于要保留的项直接将其向前覆盖到对应的位置即可.

```JavaScript
var removeElement = function(nums, val) {
    let l = 0;
    nums.forEach((el) => {
        if(el !== val)
            nums[l++] = el;
    });
    return l;
};
```

---

### [28. Implement strStr()](https://leetcode.com/problems/implement-strstr/)

返回一个子串在字符串中的起始角标. 如果不是子串返回`-1`.
字符串匹配问题, JS中有内置方法`indexOf()`. 就V8引擎来说, 其内部实现比较复杂, 对于不同长度模式串采用不同的算法, 其中对于较长的模式串使用Boyer–Moore–Horspool算法(具体可以查看[src/string-search.h](https://github.com/v8/v8/blob/master/src/string-search.h)).
常见的其他字符串匹配算法有`KMP`, `BM`和`Sunday`等, 具体可以去搜索查看相关算法.

```JavaScript
var strStr = function(haystack, needle) {
    return haystack.indexOf(needle);
};
```

---

### [29. Divide Two Integers](https://leetcode.com/problems/divide-two-integers/)

不使用乘\除\模运算, 实现两个整数(32位整型)的除法, 如果答案溢出则返回`MAX_INT`, 即`2^31 - 1`.
能够使用的运算只有位运算和加减法, 因此最基础的想法就是用减法, 看被除数中能减去多少个除数. 然而对于`2^31 - 1`除以`1`这样的输入, 直接减必定会超时, 这就需要在减的时候一次能减去多个.
一种减的思路是利用二进制, 每次尝试减去`2^i`个除数. 例如`10/2 = 5`, 实际我们可以减去的除数为`5`个, 即`2^2 + 2^0`个. 在计算除法时, 我们先用`10`减去`4 * 2`, 即`4`个除数, 余下`2`, 然后还能减去`1`个除数`2`, 因此最后答案是`4 + 1 = 5`.
对于不能整除的情况, 如`16/3 = 5`, 我们可以先减去`4`个除数, 余下`4`, 再减去`1`个除数余下`1`, 由于无法再减去任何除数, 所以得到的答案也是`5`.
但是由于无法使用乘法, 如何一次性减去`2^n`个除数呢? 这里就可以利用位移运算. 我们知道, 位运算中的左移相当于乘`2`, 右移相当于除以`2`, 因此只需要用位移就可以实现乘法了. 实际我在代码中采用的方法是, 先左移到一个足够大的值, 再每次右移一位, 依次去试减的方式. 在记录结果时, 我采用边加边左移的方式.
对于于正负数问题, 以上的讨论都是针对同号的情况(虽然只讨论了两个正数, 但是两个负数实际上是类似的), 异号的情况可以单独将正负号提出来, 直接处理同号的结果, 最后统一处理符号.
还有一个小问题在于溢出. 由于整型最小值是`-2^31`, 而最大值是`2^31 - 1`, 在遇到如`-2^31/-1`时, 结果就会超过最大值. 简单的做法就是把除数\被除数\相除结果全部处理成负数, 最后统一处理符号和溢出.
整体复杂度`O(log(N))`, `N`是相除结果.
实际上这种方法可以看做是一种二分的方法, 当然如果可以使用乘法的话, 直接二分最多能够减去多少个除数也是一种解法.

```JavaScript
var divide = function(dividend, divisor) {
    if(divisor === 0) return Number.NaN;    // 除0错误, 实际测试数据中没有这种情况
    const MAX_INT = 0x7fffffff; // 溢出时需要返回的最大值

    let isNegative = (dividend >= 0) ^ (divisor >= 0);  // 相除结果是否为负
    // 处理成负数相除
    dividend = dividend>0?-dividend:dividend;  
    divisor = divisor>0?-divisor:divisor;

    // 找到最多可以减去的除数
    let tmp = divisor;
    while(1) {
        // 如果再左移会导致溢出或者大于被除数, 则当前tmp为最大可以减去的除数
        if((tmp << 1) >= 0 || (tmp << 1) < dividend)
            break;
        tmp <<= 1;
    }
    
    let cnt  = 0;
    //  尝试减去每个2^i * divisor
    while(tmp <= divisor) {
        cnt <<= 1;
        // 判断是否够减
        if(tmp >= dividend) {
            dividend -= tmp;    // 够减则减去
            cnt -= 1;
        }
        // tmp == -1时再右移依然是-1, 直接退出避免死循环
        if(tmp == -1)
            break;
        tmp >>= 1;
    }
    cnt = isNegative?cnt:-cnt;  // 修正正负号
    return cnt>MAX_INT?MAX_INT:cnt; // 判断是否溢出, 不同编程语言判断方法不完全相同
};
```

---

### [30. Substring with Concatenation of All Words](https://leetcode.com/problems/substring-with-concatenation-of-all-words/)

给一组单词和一个字符串, 找到字符串中由这些单词连成的子串的位置. 这些单词的长度相同, 子串可以由这一组单词按一定顺序连成, 且每个单词都只用到一次. 字符串中可能有多个子串满足条件.

我们用`N`表示字符串的长度, `L`表示每个单词的长度, `K`表示单词的个数.
我们可以把原字符串看成是`L`个不同的分解串. 例如字符串`abcdefgdefabc`, `L = 3`, 则可以看成:`abc def gde fab`, `bcd efg def abc` 和 `cde fgd efa`. 其中我们将尾部多余的字符舍去. 显然, 满足答案的子串一定是这几个串中的一段. 如果我们原单词是`abc`和`def`, 分别用`A`和`B`代表, `#`表示不属于给定单词, 则这几个串可以看做`AB##`, `##BA`, `###`, 则我们只用找到其中长度为`2`的每个单词都出现过的子串.
要找到这样的子串, 我们首先要对所有单词计数, 以处理同一个单词出现多次的情况. 这个计数值表示了每个单词还有多少次可以使用. 例如, 单词`A`出现`2`次, `B`出现`1`次(`K = 3`), 我们要在`AAB#BAABBA`中找到所有满足这个条件长度为`3`的子串.
我们用一个队列记录可能合法的子串, 将一个分解串中的单词依次进入队列中. 入队单词不为`#`, 我们就在计数中减去对应单词的计数. 当队列长度达到`K`, 且都能成功减去计数, 说明对应的计数值都应该为`0`了, 此时队列中的元素组成一个合法串, 队首元素的位置就是一个答案.
同时, 为了继续判断下一个子串, 我们要将队首出队, 并将其对应的单词计数加回去, 然后再加入下一个单词继续判定.
当遇到单词`#`时, 则当前队列中任何一个单词作为开始位置, 所得到的长度为`K`的子串都会包含单词`#`, 一定不满足要求. 因此可以将所有元素出队, 并从下一个不为`#`的单词开始重新入队.
当一个计数值会被减为负数时, 说明当前队列中此单词的数量超过给出的数量, 因此队列记录的一定不是一个合法子串, 需要通过出队使其合法. 直接依此将元素出队, 出队时将出队单词的计数值加回去, 直到可以减去刚才入队单词的计数值为止.
以上面`AAB#BAABBAA`为例子. 开始`AAB`依次入队达到长度`K = 3`且都能成功减去计数, 因此为一个有效答案. 去除队首得到`AB`以后加入`#`, 所有包含`#`的都不可能是答案, 因此全部出队, 从`#`后的`BAABBAA`重新开始. 开头的`BAA`同理可以满足要求, 是一个有效答案. `B`出队后, 又加入一个`B`, 再次满足要求, 即`AAB`又是一个答案. `A`出队得到`AB`, 又加入一个`B`, 此时`B`计数已经为`0`了, 无法再减去(加入后队列中是`ABB`, 而合法串`B`只能出现一次), 因此依次出队直到队列中第一个`B`出队, 然后再将后面的`B`入队, 此时队列为`B`. 接着又能依次加入`AA`, 又得到一个答案.
由于每个分解串中的单词只会入队出队一次, 且分解串长度之和为`N`, 因此用队列找答案的复杂度为`O(N)`. 然而这里面没有考虑匹配单词的耗时. 如果使用暴力的匹配方法, 对于分解串每个单词要去匹配给定的一组单词, 复杂度退化为`O(NKL)`. 因此需要一种快速匹配的方法.
要在一个字符串中匹配一组单词可以使用AC自动机之类的数据结构, 但是这里所有单词的长度是一样的, 可以用更简单的hash来做.
我们使用前缀hash, 这样可以更快的处理出整个字符串中所有指定长度子串的hash值. 一个字符串`abc`, 则这个字符串的hash值表示为`hash('abc') = 'a' * g^2 + 'b' * g^1 + 'c' * g^0 % m`, 其中`'a'`,`'b'`,`'c'`对应字符串中的字母, 实际计算中每个字母对应一个唯一的数字, `g`和`m`都是常数, 为了减少冲突概率通常取素数.
这种hash有以下性质:
```
hash('abc') = (hash('ab') * g + 'c') % m
hash('bcd') = ((hash('abc') - ('a' * g^2) % m + m) * g + 'd') % m
```
其中第二个式子中加上`m`是为了避免出现负数.
这两个性质意味着我们可以用`O(N)`的复杂度计算得到长度为`N`的字符串中每个长度为`L`的子串的hash值. 利用第一个式子可以用`O(L)`的复杂度算得第一个长度为`L`的子串的hash, 接着用第二个式子就能用`O(1)`的复杂度得到下一个子串的hash, 因此获得所有分解串中单词的hash是`O(N)`的复杂度.
另外, 我们还需要对给出的那组单词做hash, 复杂度是`O(KL)`, 匹配时复杂度`O(1)`, 因此综合下来算法整体的复杂度是`O(N + KL)`.
在实际的代码中我为了减少冲突的可能性, 使用了双hash. 另外我用JS里面的对象作为map去计数和匹配, 但实际把`M1`和`M2`调整小一点, 直接用数组就可以.
这种方法如果遇到特殊情况使得hash冲突的话就会出错(比如给无数多个单词), 不过双hash一般很难卡掉的.

```JavaScript
var findSubstring = function(s, words) {
    if(words.length < 1 || s.length < words.length * words[0].length) {
        return [];
    }

    const M1 = 10007,
        M2 = 10009,
        G1 = 31,
        G2 = 29;
    let map = {},
        wl = words[0].length;   // 单词的长度

    // 计算每个单词的hash值
    words.forEach((word) => {
        let hash1 = 0, hash2 = 0;
        for(let i = 0; i < wl; ++i) {
            hash1 *= G1;
            hash1 += word.charCodeAt(i) - 97;
            hash1 %= M1;
            
            hash2 *= G2;
            hash2 += word.charCodeAt(i) - 97;
            hash2 %= M2;
        }
        if(map[hash1 * M2 + hash2] !== undefined)
            ++map[hash1 * M2 + hash2];
        else
            map[hash1 * M2 + hash2] = 1;
    });
    
    let topBase1 = 1,
        topBase2 = 1;
    for(let i = 1; i < wl; ++i) {
        topBase1 *= G1;
        topBase1 %= M1;
        topBase2 *= G2;
        topBase2 %= M2;
    }
    let hash1 = 0,
        hash2 = 0,
        hashArrs = new Array(wl);   // 分解串, 每个串由单词的hash值组成
    for(let i = 0; i < wl; ++i) {
        hashArrs[i] = [];
    }

    // 计算每个长度为wl的子串的hash, 并将其加入对应的分解串中
    for(let i = 0; i < s.length; ++i) {
        if(i >= wl) {
            hash1 = (hash1 - (s.charCodeAt(i - wl) - 97) * topBase1 % M1 + M1) % M1;
            hash2 = (hash2 - (s.charCodeAt(i - wl) - 97) * topBase2 % M2 + M2) % M2;
        }
        
        hash1 *= G1;
        hash1 += s.charCodeAt(i) - 97;
        hash1 %= M1;
        
        hash2 *= G2;
        hash2 += s.charCodeAt(i) - 97;
        hash2 %= M2;
        
        if(i >= wl - 1) {
            hashArrs[(i + 1) % wl].push(hash1 * M2 + hash2);
        }
    }
    
    let ret = [], ds = words.length;
    hashArrs.forEach((hashArr, idx) => {
        for(let left = 0, i = 0; i < hashArr.length; ++i) {
            if(map[hashArr[i]] !== undefined) {
                // 加入队列的单词在给出的单词中
                // 出队使得计数不会减为负数
                while(map[hashArr[i]] === 0) {
                    ++map[hashArr[left]];
                    ++left;
                }
                --map[hashArr[i]];  // 减去计数
                if(i - left + 1 === ds) {
                    // 长度满足要求, 即找到一个解
                    ret.push(left * wl + idx);
                    // 队首出队
                    ++map[hashArr[left]];
                    ++left;
                }
            } else {
                // 加入队列的单词不是给出的单词, 全部出队并从下一个单词重新开始入队
                while(left != i) {
                    ++map[hashArr[left]];
                    ++left;
                }
                ++left;
            }
            if(i === hashArr.length - 1) {
                // 搜索到分解串末尾, 全部出队还原计数值
                while(left < hashArr.length) {
                    ++map[hashArr[left]];
                    ++left;
                }
            }
        }
    });
    
    return ret;
};
```