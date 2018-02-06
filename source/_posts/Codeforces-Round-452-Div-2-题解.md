---
title: Codeforces Round 452 (Div.2) 题解
date: 2018-02-05 13:12:20
categories: 题解
tags:
  - Codeforces
  - 算法
  - 题解
---

上班摸鱼除草...
因为题解代码不是一起写的, 所以代码的逻辑和题解不完全一样, 但基本思路一致...

---

### [A. Splitting in Teams](http://codeforces.com/contest/899/problem/A)

有`n`个小组, 每组有1人或2人, 现在要将他们组成3人组, 且保证原来的2人组的人还在一起, 问最多能组成几个3人组.

乱搞题, 只有`一个1人组和一个2人组`或者`三个1人组`能组成一个3人组, 只用给每个2人组配一个1人组, 多余的1人组3人一组就行了(一个潜在条件是优先组1人组 + 2人组比直接3个1人组组队更优).

```cpp
#include<cstdio>
int main() {
    int n, x, cnt1 = 0, cnt2 = 0;
    scanf("%d", &n);
    for(int i = 0; i < n; ++i) {
        scanf("%d", &x);
        if(x == 1) ++cnt1;
        else ++cnt2;
    }
    int ans = cnt2 <= cnt1 ? cnt2 + (cnt1 - cnt2) / 3 : cnt1;
    printf("%d\n", ans);
    return  0;
}
```

---

### [B. Months and Years](http://codeforces.com/contest/899/problem/B)

给`n(1 ≤ n ≤ 24)`个整数, 每个整数代表一个月有多少天. 判断这些整数是否可以是连续的月份.

乱搞题, 闰年的2月比较特殊, 有29天, 需要一定处理. 然而, 因为最多`24`个数, 说明连续的月数不会超过2年, 每个月份出现不会超过2次, 最多只可能包含1个闰年. 我这里直接将一个闰年和两个平年各个月份的天数列出来, 做成循环数组, `平平`, `平润`, `润平`的模式都包含在了里面, 然后直接枚举看是否能完全匹配就行了.

```cpp
#include<cstdio>
int a[40] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31, 31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
int x[30], n;
bool check() {
    for(int i = 0; i < 36; ++i) {
        bool flag = true;
        for(int j = 0; j < n; ++j) {
            if(x[j] != a[(i + j) % 36]) {
                flag = false;
                break;
            }
        }
        if(flag)
            return true;
    }
    return false;
}

int main() {
    scanf("%d", &n);
    for(int i=0;i<n;++i) {
        scanf("%d", &x[i]);
    }
    printf("%s\n", check()? "Yes": "No");
}
```

---

### [C. Dividing the numbers](http://codeforces.com/contest/899/problem/C)

给`n(2 ≤ n ≤ 60 000)`个数, 分别为`1, 2, 3, ..., n`, 现在将他们分成两组, 让两组数字和之间的差距最小, 输出最小的差值和其中一组包含哪些数.

从少到多分析:
(1). 当只有`1, 2`的时候, 分为`[1]`和`[2]`两组, 差值为`1`;
(2). 当为`1, 2, 3`的时候, 分为`[1, 2]`和`[3]`, 差值为`0`;
(3). 当为`1, 2, 3, 4`的时候, 分为`[1, 4]`和`[2, 3]`, 差值为`0`;
(4). 当为`1, 2, ..., 5`时, 等价于从`(2)`再分`4, 5`, 得到`[1, 2, 4]`和`[3, 5]`, 差值为`1`;
(5). 当为`1, 2, ..., 6`时, 等价于从`(1)`再分`3, 4, 5, 6`(`3 + 6 == 4 + 5`), 得到`[1, 3, 6]`和`[2, 4, 5]`, 差值为`1`;
(6). 当为`1, 2, ..., 7`时, 等价于从`(2)`再分`4, 5, 6, 7`(`4 + 7 == 5 + 6`), 得到`[1, 2, 4, 7]`和`[3, 5, 6]`, 差值为`0`;
(7). 当为`1, 2, ..., 8`时, 等价于从`(3)`再分`5, 6, 7, 8`(`5 + 8 == 6 + 7`), 得到`[1, 4, 5 ,7]`和`[2, 3, 6, 7]`, 差值为`0`;
(8). 当为`1, 2, ..., 9`时, 等价于从`(4)`再分`6, 7, 8, 9`(`6 + 9 == 7 + 8`), 得到`[1, 2, 4, 6, 9]`和`[3, 5, 7, 8]`, 差值为`1`;
(9). 当为`1, 2, ..., 10`时, 可以从`(5)`再分`7, 8, 9, 10`;
(10). 当为`1, 2, ..., 11`时, 可以从`(6)`再分`8, 9, 10, 11`;
......
可以看出从`(5)`以后都可以以`4`为循环节往后累加, 差值与累加的基础保持一致.

恩...以上是我写题解的时候临时想的...理论AC...而之前写的时候搞得非常智障...
写的时候把差值的规律单独搞了(1, 0, 0, 1, 1, 0, 0, 1, ...), 然后把情况`(1)`, `(3)`, `(5)`, ...当成一类, `(2)`, `(4)`, `(6)`, ...当成一类, 根据`%4`以后的余数为`0`还是`2`确定最后面加哪个数...

```cpp
#include<cstdio>
void gao (int n, bool isEven) {
    int base = isEven? 0: 3;
    printf("%d\n", (n >> 1) & 1);
    if(isEven) {
        printf("%d", n >> 1);
    } else {
        printf("%d %d", 1 + (n >> 1), 3);
    }
    for(int i = 0; i < n / 4; ++i) {
        printf(" %d %d", base + i + 1, base + n - i);
    }
    if(n & 2) {
        printf(" %d\n", base + (n >> 1));
    } else {
        printf("\n");
    }
}
int main() {
    int n;
    scanf("%d", &n);
    if(n & 1) {
        gao(n - 3, false);
    } else {
        gao(n, true);
    }
}
```

---

### [D. Shovel Sale](http://codeforces.com/contest/899/problem/D)

给`n`个数`1, 2, 3, ..., n`, 其中$n(2 ≤ n ≤ 10^9)$. 取两个数让两数之和结尾的9尽量多, 问有多少种取法.

我的思路是先确定两个数之和结尾最多能有多少个`9`, 然后再计算有多少种方案.
1. 当`n < 5`时, 无论如何组合都不能形成`9`, 因此数量为`0`, 方案是`n`个数中任意取`2`个的方案数;
2. 如果`n + (n - 1)`每一位都是`9`, 这样的数只可能得到`1`个, 因此方案数是`1`;
3. 如果`n + (n - 1)`除了第一位为`x`, 每一位都是`9`, 则需要确认`x9...9`, `(x-1)9...9`, ..., `09...9`每种情况有多少方案.
4. 其他情况则需要确认`(x-1)9...9`, ..., `09...9`每种情况有多少方案.

这样问题就简化成对于每种情况`z`, 有多少满足`x + y = z`, 其中`x < y`且在`1, 2, ..., n`中. 我们只用确定每种情况下`x`或者`y`的有效取值范围就能知道有多少方案.

```cpp
#include<cstdio>
int main() {
    int n;
    scanf("%d", &n);
    if(n < 5) {
        printf("%d\n", n * (n - 1) / 2);
        return 0;
    }
    int sum = n + n - 1;
    bool flag = true, ends = true;
    int cur, cnt = 0;
    while(sum) {
        cur = sum % 10;
        if(!flag) {
            ends = false;
        }
        if(cur != 9) {
            flag = false;
        }
        ++cnt;
        sum /= 10;
    }
    if(flag) {
        printf("1\n");
        return 0;
    }
    --cnt;
    if(!ends)
        --cur;
    int top = 1;
    for(int i = 0; i < cnt; ++i) {
        cur *= 10;
        top *= 10;
        cur += 9;
    }
    int ans = 0;
    while(cur >= 0) {
        int max = cur > n ? n : cur - 1;
        ans += max - cur/2;
        cur -= top;
    }
    printf("%d\n", ans);
    return 0;
}
```

---

### [E. Segments Removal](http://codeforces.com/contest/899/problem/E)

`n`个数的序列, 其中$n(1 ≤ n ≤ 200000)$, 每个数$a_i(1 ≤ a_i ≤ 10^9)$. 每次移除连续相同数字最多的一段, 如果两段相同数字数量一致, 则移除靠左的一段. 问经过几次操作能移除所有数字.

这题唯一的复杂的点在于移除一段数字后可能原先是两段的数字会合成一段. 例如`aabbbaa`这种形式, 移除`bbb`后剩余`aaaa`变成了一段, 只需要`2`次就能完全移除.

整个流程没有什么分治策略或者其他规律, 似乎只能模拟这个流程. 然而直接的模拟复杂度为`O(n^2)`, 因此判断要用某种数据结构将复杂度降低到`O(nlog(n))`.
对于连续的一段相同数字, 我们可以将其看作一个节点, 例如`aabbbaa`可看成`a2`,`b3`,`a2`三个节点. 既然要方便的移除节点, 而且要知道左右邻居, 很自然的能想到双向链表结构.
那么如何快速查找要移除的节点呢? 同时由于节点可能合并, 因此节点的移除顺序很难预先排定, 这样就需要可以动态调整的结构, 我这里决定使用二叉堆(其实直接用stl的优先队列就行了, 但是队列需要存节点的引用).
我们将连续数字更多, 更靠左的节点放在优先队列头部, 当这个节点出队时, 同时将其在链表中摘除. 同时需要判断摘除后左右两个节点是否能合并, 如果能合并的话需要生成一个新的合并后的节点放入链表, 并将合并前的两个节点从链表和优先队列里移除. 当然也可以只移除一个, 然后更新另一个节点数据达到同样的效果. 我代码中没有直接将节点从队列中移除, 而是做了个标记, 当有标记的节点出队时不做任何处理.
另一个可能造成困扰的地方在于节点的位置更新. 例如`a2b3c2`, 原始位置分别是`a2(0)`, `b3(1)`, `c2(2)`, 移除`b3`后变成`a2(0)`, `c2(1)`. 如果移除后更新其后每个节点的位置, 复杂度又会提高. 然而这里只用确保相对位置不变就行了, 不更新位置信息对优先级没有影响.

从链表移除复杂度`O(1)`, 插入优先队列和从优先队列移除复杂度`O(log(n))`, 总共不超过`n`个节点, 复杂度为`O(nlog(n))`.

```cpp
#include <cstdio>
#include <iostream>
using namespace std;
struct Node {
    int val, cnt, l, r, p;
    bool drop = false;
}node[400002];
int heap[200001];
int np = 0, hp = 0;
void setNode (int i, int val, int cnt, int l, int r, int p) {
    node[i].val = val;
    node[i].cnt = cnt;
    node[i].l = l;
    node[i].r = r;
    node[i].p = p;
}
void push(int idx) {
    heap[hp] = idx;
    int p = hp++;
    while(p) {
        int f = (p - 1) / 2;
        int fi = heap[f];
        if(node[fi].cnt > node[idx].cnt || (node[fi].cnt == node[idx].cnt && node[fi].p < node[idx].p)) {
            break;
        }
        heap[f] = idx;
        heap[p] = fi;
        p = f;
    }
}
void pop() {
    if(!hp)
        return;
    heap[0] = heap[--hp];
    int p = 0;
    int pi = heap[p];
    while(p * 2 + 1 < hp) {
        int c, ci;
        if(p * 2 + 2 < hp) {
            int c1 = p * 2 + 1;
            int c2 = c1 + 1;
            int ci1 = heap[c1], ci2 = heap[c2];
            if(node[ci1].cnt > node[ci2].cnt || (node[ci1].cnt == node[ci2].cnt && node[ci1].p < node[ci2].p)) {
                c = c1;
                ci = ci1;
            } else {
                c = c2;
                ci = ci2;
            }
        } else {
            c = p * 2 + 1;
            ci = heap[c];
        }
        if(node[pi].cnt > node[ci].cnt || (node[pi].cnt == node[ci].cnt && node[pi].p < node[ci].p)) {
            break;
        }
        heap[p] = ci;
        heap[c] = pi;
        p = c;
    }
}
int top () {
    if(!hp)
        return -1;
    return heap[0];
}
int main() {
    int n, v, cnt, x;
    scanf("%d", &n);
    for(int i = 0; i < n; ++i) {
        if(i == 0) {
            scanf("%d", &v);
            cnt = 1;
            continue;
        }
        scanf("%d", &x);
        if(x == v) {
            ++cnt;
            continue;
        }
        setNode(np, v, cnt, np - 1, np + 1, np);
        ++np;
        cnt = 1;
        v = x;
    }
    setNode(np, v, cnt, np - 1, -1, np);
    ++np;
    for(int i = 0; i < np; ++i) {
        push(i);
    }
    int ans = 0;
    while(hp) {
        int p = top();
        pop();
        if(node[p].drop) {
            continue;
        }
        ++ans;
        int l = node[p].l;
        int r = node[p].r;
        if(~r) {
            node[r].l = l; 
        }
        if(~l) {
            node[l].r = r;
        }
        if(~l && ~r && node[l].val == node[r].val) {
            setNode(np, node[l].val, node[l].cnt + node[r].cnt, node[l].l, node[r].r, node[l].p);
            if(~node[r].r) {
                node[node[r].r].l = np;
            }
            if(~node[l].l) {
                node[node[l].l].r = np;
            }
            push(np);
            ++np;
            node[l].drop = node[r].drop = true;
        }
    }
    printf("%d\n", ans);
    return 0;
}
```

---

### [F. Letters Removing](http://codeforces.com/contest/899/problem/F)

给一条长度为`n`(由大小写字母或数字组成)的字符串和`m`个操作, 其中$(1 ≤ n, m ≤ 2·10^5)$, 每个操作包含`(l, r, c)`, 其中`l`和`r`表示字符串的一段区间, `c`表示要移除的字符. 输入保证每组`(l, r)`都是当前字符串的有效区间, 求操作完成后剩下的字符串.

由于操作间会互相影响, 前一次操作后的结果决定后一次操作的结果, 很难有什么方法来合并操作, 也不能改变操作顺序. 因此考虑需要模拟完成这些操作.
由于每次移除后会对后面的字符串位置有影响, 进而影响到下一次移除操作的作用范围, 我们需要用某种操作来维护这些信息, 以便确认下一次操作的作用范围. 然而字符串长度较长, 每次操作都将每个字符位置更新到是不可能的, 因此需要一个操作来记录一段的更新, 就像线段树的懒操作一样.
再这样的考虑下, 我使用线段树来记录整个字符串, 每次更新时使用懒操作在对应区间节点做一个记录, 在需要查询其子节点时再将懒操作更新下去.
由于删除会改变字符的位置, 线段树节点表示的范围并不是真实的范围, 而是在原字符串的范围. 为了将操作的范围映射到正确的节点, 我们需要记录每个节点下还剩余多少字符, 这样我们可以通过节点的左儿子包含字符的个数确定当前操作范围影响到左儿子的范围有多少.
同时, 为了快速确定移除操作后删除了多少个字符, 需要对每个字符做个数统计. 如果对每个可能的字符都开数组记录估计会超内存, 我这里直接用了map, 包括记录历史操作也是用的map.
最后只要遍历一遍树的叶子, 把所有操作更新下去就可以得到最终的结果了.

线段树构建和遍历需要走一遍节点, 复杂度是`O(n)`, 更新操作的时间是`O(log(n))`, 需要更新`m`次, 整体复杂度`O(mlog(n) + n)`.

```cpp
#include<cstdio>
#include<cstring>
#include<map>
using namespace std;

struct Node {
    map<char, int> cmap;
    map<char, bool> history;
    int l = -1, r = -1, cnt = 0;
} node[800001];
int l = 0;

void init (int p, char* str, int len) {
    if(len == 1) {
        node[p].cnt = 1;
        node[p].cmap[str[0]] = 1;
        return;
    }
    int mid = len / 2;
    node[p].l = l++;
    init(node[p].l, str, mid);
    node[p].r = l++;
    init(node[p].r, str + mid, len - mid);
    node[p].cnt = len;
    for(auto it = node[node[p].l].cmap.begin(); it != node[node[p].l].cmap.end(); ++it) {
        node[p].cmap[it->first] += it->second;
    }
    for(auto it = node[node[p].r].cmap.begin(); it != node[node[p].r].cmap.end(); ++it) {
        node[p].cmap[it->first] += it->second;
    }
}

int remove (int p, int l, int r, char c) {
    int cnt = 0;
    if(!~p || !node[p].cmap[c]) {
        return 0;
    }
    if(r - l + 1 == node[p].cnt) {
        cnt = node[p].cmap[c];
        node[p].cnt -= cnt;
        node[p].cmap[c] = 0;
        if(node[p].cnt) {
            node[p].history[c] = true;
        } else {
            node[p].l = node[p].r = -1;
            node[p].history.clear();
        }
        return cnt;
    }
    if(node[p].history.size()) {
        for(auto it = node[p].history.begin(); it != node[p].history.end(); ++it) {
            if(~node[p].l) {
                remove(node[p].l, 1, node[node[p].l].cnt, it -> first);
            }
            if(~node[p].r) {
                remove(node[p].r, 1, node[node[p].r].cnt, it -> first);
            }
        }
        node[p].history.clear();
    }
    int lcnt = 0;
    if(~node[p].l) {
        lcnt = node[node[p].l].cnt;
    }
    if(lcnt >= r) {
        cnt += remove(node[p].l, l, r, c);
    } else if (lcnt < l) {
        cnt += remove(node[p].r, l - lcnt, r - lcnt, c);
    } else {
        cnt += remove(node[p].l, l, lcnt, c);
        cnt += remove(node[p].r, 1, r - lcnt, c);
    }
    node[p].cnt -= cnt;
    node[p].cmap[c] -= cnt;
    return cnt;
}

void dfs(int p) {
    if(!~p || !node[p].cnt) {
        return;
    }
    if(node[p].cnt == 1) {
        for(auto it = node[p].cmap.begin(); it != node[p].cmap.end(); ++it) {
            if(it -> second)
                printf("%c", it -> first);
        }
        return;
    }
    if(node[p].history.size()) {
        for(auto it = node[p].history.begin(); it != node[p].history.end(); ++it) {
            if(~node[p].l) {
                remove(node[p].l, 1, node[node[p].l].cnt, it -> first);
            }
            if(~node[p].r) {
                remove(node[p].r, 1, node[node[p].r].cnt, it -> first);
            }
        }
        node[p].history.clear();
    }
    dfs(node[p].l);
    dfs(node[p].r);
}

int main() {
    int n, m, len, x, y;
    char c, str[200001];
    scanf("%d %d", &n, &m);
    scanf("%s", str);
    // printf("%d %d %s\n", n, m, str);
    len = strlen(str);
    init(l++, str, len);
    for(int i = 0; i < m; ++i) {
        scanf("%d %d %c", &x, &y, &c);
        remove(0, x, y, c);
    }
    dfs(0);
    return 0;
}
```
