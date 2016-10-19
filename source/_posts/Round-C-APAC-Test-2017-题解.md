---
title: Round C APAC Test 2017 题解
date: 2016-10-15 14:51:37
categories: 题解
tags:
  - APAC
  - 算法
  - 题解
---

Round D都要开始了, Round C还没搞定的我(D题不会搞)...另外300+名次也能拿到面试资格呢, 然而做的人却越来越少了.

---

### [Problem A. Monster Path](https://code.google.com/codejam/contest/6274486/dashboard#s=p0)

neta某手游...给个`R*C`的矩形地图, 上面用`.`和`A`标注, 走到`A`上有`P`的概率抓到妖怪, 走到`.`上有`Q`的概率抓到妖怪. 当在一个位置抓过妖怪, 下次经过就抓不到了, 给定起点坐标`Rs`和`Cs`, 需要走一条路线使得走`S`步获得妖怪数的期望最大, 求这个期望.

因为`S`最大只有`9`, 直接乱dfs每条路线就行了. 用数组记录一下当前位置没有抓过怪物的概率, 然后搜的时候每经过一个地点, 可计算出走这里抓到怪物的概率, 然后更新数组里当前位置没有抓过怪物的概率, 继续搜下去, 过程中记录一下概率和就是期望.

```cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
    freopen((f+".in").c_str(),"r",stdin);
    freopen((f+".out").c_str(),"w",stdout);
}
int r,c,x,y,s;
double p,q;
bool grid[22][22];
double pro[22][22];
int step[4][2] = {0,1,1,0,0,-1,-1,0};
double ans;
void dfs(int x, int y, int st, double cnt) {
    int tx,ty;
    double tp, tmp;
    if(st!=s) {
        tp = grid[x][y]?pro[x][y]*p:pro[x][y]*q;
    } else {
        tp = 0;
    }
    if(!st) {
        ans = max(ans, cnt+tp);
        return;
    }
    for(int i=0;i<4;++i) {
        tx = x + step[i][0];
        ty = y + step[i][1];
        if(tx>=0&&tx<r&&ty>=0&&ty<c) {
            pro[x][y] -= tp;
            dfs(tx, ty, st - 1, cnt+tp);
            pro[x][y] += tp;
        }
    }
}
void gao(){
    scanf("%d%d%d%d%d",&r,&c,&x,&y,&s);
    scanf("%lf%lf",&p,&q);
    char a,b;
    scanf("%c", &b);
    for(int i=0;i<r;++i) {
        for(int j=0;j<c;++j) {
            scanf("%c%c",&a,&b);
            if(a=='A')
                grid[i][j] = 1;
            else
                grid[i][j] = 0;
            pro[i][j] = 1;
        }

    }
    ans = 0;
    dfs(x,y,s,0);
    printf("%.7f\n", ans);
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

### [Problem B. Safe Squares](https://code.google.com/codejam/contest/6274486/dashboard#s=p1)

为啥我觉得这题比较难, 过的人却最多...
给一个`R*C`的矩形, 给`K`个矩形上的点, 求矩形上有多少个正方形不包含这些点中任意一个.

由于`R`,`C`,`S`最大能到`3000`, 所以需要向`O(RC)`上靠, 自然会想到动态规划.
如果用`dp[i][j]`表示前面`i*j`的矩形中包含的正方形数, 则`dp[i-1][j] + dp[i][j-1] - dp[i-1][j-1]`表示前面`i*j`矩形中不含当前点`grid[i][j]`的正方形数量. 为了求得`dp[i][j]`, 还需要加上所有以`grid[i][j]`为右下角的正方形数量.
因为光枚举`i`和`j`复杂度就接近`10^7`, 因此需要预处理以`grid[i][j]`为右下角的正方形数量.
以`grid[i][j]`为右下角的正方形数量等于以`grid[i][j]`为右下角的最大合法正方形边长(从`1`开始取, 每个长度都能找到一个合法的正方形), 问题变成了找上方和左方最近的非法点的距离. 这个依然可以用动态规划做.
如果`dpDis[i][j]`表示以`grid[i][j]`为右下角的最大合法正方形边长, 则`dpDis[i][j]`要么被最接近的非法的`grid[i-x][j]`限制, 要么被最接近的非法的`grid[i][j-x]`限制, 要么和`dpDis[i-1][j-1]`受到相同的非法点限制. 我们用`left[i][j]`表示最接近的左边的非法点距离, `top[i][j]`表示最接近的上边的非法点距离, 则有:
`dpDis[i][j] = min(top[i][j], left[i][j], dpDis[i-1][j-1])`.
其中`left[i][j]`可以很容易根据`left[i-1][j]`和当前位置点的合法性计算出来, `top[i][j]`同理.
有了`dpDis[i][j]`之后我们就可以完成之前的公式:
`dp[i][j] = dp[i-1][j] + dp[i][j-1] - dp[i-1][j-1] + dpDis[i][j]`.
复杂度`O(RC)`.

```cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
    freopen((f+".in").c_str(),"r",stdin);
    freopen((f+".out").c_str(),"w",stdout);
}

bool grid[3010][3010];
ll dp[3010][3010];
ll rbcnt[3010][3010];
ll lt[3010][3010][2];
int r,c,k;
void gao(){
    int x,y;
    scanf("%d%d%d",&r,&c,&k);
    memset(grid,0,sizeof(grid));
    memset(dp,0,sizeof(dp));
    memset(rbcnt,0,sizeof(rbcnt));
    memset(lt,0,sizeof(lt));
    for(int i=0;i<k;++i) {
        scanf("%d%d",&x,&y);
        grid[x+1][y+1] = 1;
    }
    for(int i=1;i<=r;++i) {
        for(int j=1;j<=c;++j) {
            lt[i][j][0] = grid[i][j]?0:lt[i-1][j][0]+1;
            lt[i][j][1] = grid[i][j]?0:lt[i][j-1][1]+1;
            rbcnt[i][j] = grid[i][j]?0:min(rbcnt[i-1][j-1]+1, min(lt[i][j][0],lt[i][j][1]));
            dp[i][j] = dp[i-1][j] + dp[i][j-1] - dp[i-1][j-1] + rbcnt[i][j];
        }
    }
    printf("%lld\n", dp[r][c]);
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

另一种计算`dpDis[i][j]`的方法是通过二分.
用数字`1`表示非法点, `0`表示合法点, 预处理数组`sum[i][j]`表示矩形中`1`的个数.
有了`sum[i][j]`数组, 可以得到其中任意正方形中`1`的个数. 如要得到以`grid[i][j]`为右下角, 边长为`L`的正方形内`1`的个数, 可以用`sum[i][j] - sum[i-l][j] - sum[i][j-l] + sum[i-l][j-l]`求得, 然后通过二分, 查找使得`1`个数为`0`的最大边长`L`, 就是`dpDis[i][j]`的值.

---

### [Problem C. Evaluation](https://code.google.com/codejam/contest/6274486/dashboard#s=p2)

给`N`个形如`a=f(b,c)`的字符串, 表示变量`a`的值依赖`b`和`c`的值. `b=g()`这样的字符串表示给`b`一个基本值. `N`不超过`1000`, 左侧变量只会在左侧出现一次, 右侧函数内变量数不超过`10`. 现在可以任意安排顺序, 问这些字符串能否使得每个变量都能够求得值(是否合法).

直接把每个变量名拿出来(想念js正则), 依赖关系用单向边连起来, 构成图. 如`a`依赖`b`和`c`,则用`c`和`b`指向`a`. 然后我们根据入度关系, 用拓扑序进行更新, 从有值的点向依赖他的点更新, 最后看看是否有入度不为`0`的点(依赖成环)或者没有定值的点(如没有定义基本值). 复杂度为`O(N)`(点数不会超过`11*N`, 边数也不会超过`10*N`).

看代码的时候需要注意的是, 我代码里面本意是用`val[]`表示是否能求出值, 但实际上只要其依赖的变量中有一个能求出值, 其`val[]`就会变成`true`. 其需要入度计数`cnt[]`辅助, 当`val[]`为`true`且入度为`0`时才保证其依赖的数都已经有值了.

```cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
    freopen((f+".in").c_str(),"r",stdin);
    freopen((f+".out").c_str(),"w",stdout);
}
int n;
bool val[10010];
int cnt[10010];
vector<int> nxt[10010];

void gao(){
    scanf("%d",&n);
    int l = 0;
    map<string, int> ma;
    string str;
    memset(val,0,sizeof(val));
    memset(cnt,0,sizeof(cnt));
    for(int i=0;i<10010;++i) {
        nxt[i].clear();
    }
    for(int i=0;i<n;++i) {
        cin>>str;
        string tmp;
        int lf;
        int j=0;
        for(;str[j]!='=';++j) {
            tmp.push_back(str[j]);
        }
        if(!ma[tmp]) {
            lf = ++l;
            ma[tmp] = l;
        } else {
            lf = ma[tmp];
        }
        while(str[j]!='(')
            ++j;
        while(str[j]!=')') {
            ++j;
            tmp.clear();
            while(str[j]!=','&&str[j]!=')') {
                tmp.push_back(str[j]);
                ++j;
            }
            if(tmp.length()==0) {
                val[lf] = 1;
            } else {
                if(!ma[tmp]) {
                    ma[tmp] = ++l;
                }
                nxt[ma[tmp]].push_back(lf);
                ++cnt[lf];
            }
        }
    }

    queue<int> qu;
    for(int i=1;i<=l;++i) {
        if(val[i]&&!cnt[i]) {
            qu.push(i);
        }
    }
    while(!qu.empty()) {
        int x = qu.front();
        qu.pop();
        for(int i=0;i<nxt[x].size();++i) {
            int y = nxt[x][i];
            --cnt[y];
            val[y] = 1;
            if(!cnt[y])
                qu.push(y);
        }
    }
    bool ans = true;
    for(int i=1;i<=l;++i) {
        if(cnt[i]||!val[i]) {
            ans = false;
            break;
        }
    }
    if(ans)
        printf("GOOD\n");
    else
        printf("BAD\n");

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

### [Problem D. Soldiers](https://code.google.com/codejam/contest/6274486/dashboard#s=p3)

彻底不会..看了份代码才想明白. 想到了就是SB题, 可惜智商不够..
给`N`个士兵, 每个有一个攻击力`A[i]`和防御力`D[i]`, Alice和Bob轮流选且Alice先选, 每次选择的士兵攻击力或者防御力必须大于所有已经被选中的士兵, 直到不能选. 两人都身经百战, 见得多了, 问Alice是否能够比Bob多选一个士兵.

Alice多选一个士兵等价于先手走最后一步.
假设攻击力最高是`x`, 防御力最高是`y`, 可知当选择的士兵里有攻击力为`x`和防御力为`y`的士兵时就不能再选了.
如果有一个士兵的攻击力为`x`防御力为`y`, 则直接选他, 先手必胜.
如果没有这样的士兵, 则只有攻击力为`x`且防御力小于`y`的士兵和防御力为`y`且攻击力小于`x`. 对于这两类士兵, 每类至少有一个士兵, 先选到其中一个的人必败, 因为后手可以选另一类的任意一个使得先手无法再继续选择. 因此要避免先选`A[i] = x`且`D[i] < y`和`A[i] < x`且`D[i] = y`的士兵.
要避免这样的选择, 即要在`A[i] < x`且`D[i] < y`的士兵集合中走最后一步, 就形成了子问题.
整体复杂度`O(n^2)`.

```cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
    freopen((f+".in").c_str(),"r",stdin);
    freopen((f+".out").c_str(),"w",stdout);
}
int a[4010],d[4010],n;
bool dfs() {
    if(n == 0) {
        return 0;
    }
    int x = 0,y = 0;
    for(int i=0;i<n;++i) {
        x = max(x, a[i]);
        y = max(y, d[i]);
    }
    int l = 0;
    for(int i=0;i<n;++i) {
        if(a[i]!=x&&d[i]!=y) {
            a[l] = a[i];
            d[l] = d[i];
            ++l;
            continue;
        }
        if(a[i]==x&&d[i]==y)
            return 1;
    }
    n = l;
    return dfs();
}
void gao(){
    scanf("%d",&n);
    for(int i=0;i<n;++i) {
        used[i] = 0;
        scanf("%d%d",&a[i],&d[i]);
    }
    if(dfs()) {
        printf("YES\n");
        return;
    }
    printf("NO\n");
    return;
}
int main()
{
    useFile("D-large-practice");
    int t;
    scanf("%d",&t);
    for(int i=1;i<=t;++i) {
        printf("Case #%d: ",i);
        gao();
    }
    return 0;
}

```
