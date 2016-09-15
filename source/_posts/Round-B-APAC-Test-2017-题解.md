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

复杂度`O(1)`.

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

给定`A`, `B`, `N`和`K`, 问有多少组`(i, j)`满足$i^A + j^B$能被`K`整除. 其中`i`和`j`为不大于`N`的正整数, 且`i`与`j`不能相等. 这里`N`的范围较大, Large下$1 ≤ N ≤ 10^{18}$, 而$1 ≤ K ≤ 100000$.

$i^A + j^B$能被`K`整除意味着$i^A + j^B mod K = 0$, 其实就只用考虑$i^A mod K$和$j^B mod K$的值就行了. 对于$i^A mod K$的`0`到`K-1`每种取值计数, 然后用$j^B mod K$的取值去查询`i`满足整除条件的计数. 由于`N`很大, 不能枚举, 我们需要在`K`的范围内枚举. 由于$(K+1)^A mod K = 1^A mod K$, 对于$i^A mod K$的计数, 我们可知:
如果$i ≤ N mod K$, 计数为`N/K + 1`, 否则为`N/K`.
这样就把`i`的枚举范围缩小到`K`.

当然, 还需要用快速幂计算$i^A mod K$以及去掉`i`和`j`相同的情况.

复杂度`O(K*log(max(A, B)))`(我代码里用了map, 复杂度会比这个更高一些).

```cpp
//省略头文件
typedef long long ll;
void useFile(string f) {
    freopen((f+".in").c_str(),"r",stdin);
    freopen((f+".out").c_str(),"w",stdout);
}
ll a, b, n, k;
ll M = 1000000007;
ll fastPow(ll base, ll x) {
    ll ret = 1;
    while(x) {
        if(x&1) {
            ret*=base;
            ret%=k;
        }
        base*=base;
        base%=k;
        x>>=1;
    }
    return ret%k;
}
void gao(){
    scanf("%lld%lld%lld%lld", &a, &b, &n, &k);
    map<ll, ll> ma;
    ll cnt = n/k;
    ll om = n%k;
    ll tmp;
    for(ll i=1;i<=k;++i) {
        tmp = fastPow(i, a);
        if(i<=om) {
            ma[tmp] += (cnt+1)%M;
        } else {
            ma[tmp] += cnt%M;
        }
        ma[tmp] %= M;
    }
    ll ans = 0;
    for(ll i=1;i<=k;++i) {
        tmp = (k - fastPow(i, b))%k;
        if(i<=om) {
            if(fastPow(i, a) == tmp) {
                ans += ma[tmp]*((cnt+1)%M) - (cnt+1)%M;
                ans = (ans%M+M)%M;
            } else {
                ans += ma[tmp]*((cnt+1)%M);
                ans %= M;
            }
        } else {
            if(fastPow(i, a) == tmp) {
                ans += ma[tmp]*(cnt%M) - cnt%M;
                ans = (ans%M+M)%M;
            } else {
                ans += ma[tmp]*(cnt%M);
                ans %= M;
            }
        }
    }
    printf("%lld\n", ans);
}
int main()
{
    useFile("B-large-practice");
    int t;
    scanf("%d",&t);
    for(int i=1;i<=t;++i) {
        printf("Case #%d: ",i);
        gao();
    }
    return 0;
}
```

---

### [C. Watson and Intervals](https://code.google.com/codejam/contest/5254487/dashboard#s=p2)

给一堆整数可以用来计算`N`个线段, 线段都是闭区间, 问用其中`N-1`个线段最少覆盖多少个整数点. 其中$1 ≤ N ≤ 500000$.

乍一看是线段树, 然而这么短时间去写一颗线段树, 你已经输了.

根据公式可以计算出所有的线段, 然后将这些线段按左端点顺序从左到右排序(左端点相同按右端点从左到右排), 这样我们可以通过维护前面线段覆盖到最右的位置去计算所有线段的覆盖范围.
比如前面的线段覆盖到位置`x`, 当前线段要覆盖`l`到`r`. 那么:

1. 如果`r ≤ x`则说明前面的线段能够完全覆盖当前线段(因为排序了左端点的缘故), 当前线段不会增加覆盖范围;
2. 如果`x < l`说明当前线段和之前的线段覆盖范围不相交, 所以增加的覆盖范围是`r - l + 1`;
3. 如果上面两条不满足, 则两者有部分相交, 增加的范围是`r - x`.

这样我们枚举去掉的那个线段, 求其他线段的覆盖范围, 就能搞出Small.
而要处理Large, 我们可以先计算所有线段的覆盖大小, 再减去每个线段范围里只有他自己覆盖一次的范围中最大的那个. 为了计算每个线段仅有它自己覆盖的范围, 我们可以通过"减法"的方式实现.

1. 如果当前线段前的线段与当前线段有重合, 那一定能完整覆盖当前线段左侧. 排序后前面的线段左端比当前线段靠前, 因此当前线段的左端一定落在前面线段的覆盖范围中.
2. 对于后面的线段, 如果右端点在当前线段右端点之后, 则说明这个线段已经覆盖了当前线段的最右侧, 之后的线段不能再覆盖掉更大范围了. 也就是说, 我们计算减法的时候, 只用计算到满足这个条件的第一个线段即可.
3. 对于当点线段和2中所说的线段之前的线段, 求其覆盖范围, 然后从当前线段的范围里面减去就行. 此外如果删除这些线段中的一个, 减少的值一定为`0`, 因为当前线段能覆盖他们的覆盖范围. 也就是说, 做减法需要枚举的这些线段不会是答案所要删除的线段.

综上, 可以直接线性的计算出删除一个线段会减少的覆盖数, 只需要更新当前线段的未被前面线段覆盖的范围, 并从范围内减去后面线段重复覆盖的范围即可, 后面的线段要么可以跳过(完全在其区间内, 减少值一定为0), 要么可以作为下一个当前线段.

计算出删除每个段减少的覆盖范围后, 找出最大的值, 用总覆盖减去这个值就可以了.

复杂度`O(N*log(N))`

```cpp
//省略头文件
typedef long long ll;
void useFile(string f) {
    freopen((f+".in").c_str(),"r",stdin);
    freopen((f+".out").c_str(),"w",stdout);
}
int n;
ll l, r, a, b, c1, c2, m;
struct Node {
    ll x,y,m;
    bool operator <(Node &a) const {
        return (x==a.x?y<a.y:x<a.x);
    }
}node[500010];
void gao(){
    scanf("%d%lld%lld%lld%lld%lld%lld%lld", &n, &l, &r, &a, &b, &c1, &c2, &m);
    node[0].x = l;
    node[0].y = r;
    for(int i=1;i<n;++i) {
        Node &bf = node[i-1];
        Node &cur = node[i];
        cur.x = (a*bf.x + b*bf.y + c1)%m;
        cur.y = (a*bf.y + b*bf.x + c2)%m;
    }
    for(int i=1;i<n;++i) {
        Node &cur = node[i];
        if(cur.x>cur.y) {
            swap(cur.x, cur.y);
        }
        cur.m = 0;
    }
    sort(node, node+n);
    ll ans = 0, last = -1;
    for(int i=0;i<n;++i) {
        if(node[i].y <= last)
            continue;
        if(node[i].x > last) {
            ans += node[i].y - node[i].x+1;
        } else {
            ans += node[i].y - last;
        }
        last = node[i].y;
    }
    last = node[0].x - 1;
    node[0].m = node[0].y - node[0].x + 1;
    int id = 0;
    for(int i=1;i<n;++i) {
        Node &bf = node[id];
        Node &cur = node[i];
        ll lb, rb;
        if(cur.y <= bf.y) {
            cur.m = 0;
            lb = max(last+1, cur.x);
            rb = cur.y;
            if(rb>=lb)
                bf.m -= rb - lb + 1;
            last = max(cur.y, last);
        } else if(cur.x > bf.y) {
            cur.m = cur.y - cur.x + 1;
            id = i;
            last = cur.x - 1;
        } else {
            lb = max(last+1, cur.x);
            rb = bf.y;
            bf.m -= rb - lb + 1;
            cur.m = cur.y - bf.y;
            last = bf.y;
            id = i;
        }
    }
    ll mtmp = 0;
    for(int i=0;i<n;++i) {
        mtmp = max(mtmp, node[i].m);
    }
    printf("%lld\n", ans-mtmp);
}
int main()
{
    useFile("C-large-practice");
    int t;
    scanf("%d",&t);
    for(int i=1;i<=t;++i) {
        printf("Case #%d: ",i);
        gao();
    }
    return 0;
}
```

---

### [D. Sherlock and Permutation Sorting](https://code.google.com/codejam/contest/5254487/dashboard#s=p3)

定义一个操作`f`, 表示对一个排列进行分块, 能分的最大块数. 分块的依据是: 数字间位置不变, 前一个块的每个数都要小于后一个块的数字. 如`f([1,2,3])`为`3`, 可以分成`[1][2][3]`, 而`f([2,1,3])`为`2`, 因为只能分成`[2,1][3]`.
现在给定`N`和`M`, 对于`N`个数字的每种排列求最大分块数, 然后求出他们的平方和模上`M`.

我只想出一种`O(N^3)`的动态规划做法, 由于case数也达到`100`, 也只能跑跑Small, 复杂度到`O(10^8)`, 还好是离线跑, 还是挺快的.

对于一个排列`[a,b,c,d,e]`, 如果一定要在`c`分块, 那么`a,b,c`一定对应`1,2,3`的一个排列, 而且如果要让`[a,b,c]`不能再分, 那他们只能对应一个分块数必须为`1`的排列. 剩下的`[d,e]`对应`[4,5]`, 对其分块结果等价于对`[1,2]`分块.
现在我们用`dp[i][j]`表示长度为`i`的排列最大分成`j`块的有多少种. 那么我们可以枚举第一个分块的点`k`, 这样就有转移:
`dp[i][j] = Sum(dp[k][1]*dp[i-k][j-1])`
即前`k`个分一块的种类与后`i-k`个分`j-1`块的种类相组合.
分`1`块的种数可以用总的排列数减去其他的数量得到.
这样`dp`完成后, 枚举每种分块数对应的种数, 直接计算答案就可以了.

```cpp
//省略头文件
typedef long long ll;
void useFile(string f) {
    freopen((f+".in").c_str(),"r",stdin);
    freopen((f+".out").c_str(),"w",stdout);
}
ll dp[110][110];
ll a[110];
ll n,m;
void init() {
    memset(dp,0,sizeof(dp));
    dp[1][1] = 1;
    a[1] = 1;
    for(int i=2;i<=n;++i) {
        a[i]=a[i-1]*i;
        a[i]%=m;
        ll tmp = 0;
        for(int j=1;j<i;++j) {
            for(int k=1;k<=j;++k) {
                dp[i][k+1] += dp[i-j][1]*dp[j][k];
                dp[i][k+1]%=m;
                tmp += dp[i-j][1]*dp[j][k];
                tmp%=m;
            }
        }
        dp[i][1] = (a[i] - tmp)%m;
        dp[i][1] = (dp[i][1]+m)%m;
    }
}
void gao(){
    scanf("%lld%lld",&n,&m);
    init();
    ll ans = 0;
    for(int i=1;i<=n;++i) {
        ans += i*i%m*dp[n][i]%m;
        ans%=m;
    }
    printf("%lld\n", ans);
}
int main()
{
    useFile("D-small-practice");
    int t;
    scanf("%d",&t);
    for(int i=1;i<=t;++i) {
        printf("Case #%d: ",i);
        gao();
    }
    return 0;
}
```