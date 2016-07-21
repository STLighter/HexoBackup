---
title: Round A APAC Test 2017
date: 2016-07-10 15:55:11
categories: 题解
tags:
  - APAC
  - 算法
  - 题解
---

据说这次过了下轮就不能参加了呢,遗憾...查重结束, 可以贴代码出来了.

---

### [A. Country Leader](https://code.google.com/codejam/contest/11274486/dashboard#s=p0)

选领导人,名字里面字母种类多的当选,如果有多人名字种类一样多,则选名字字典序靠前的.给你$N$个名字$N ≤ 100$,问那个人会当选.所有的名字只有大写字母,长度不超过$20$,大数据的名字会有空格.

这题就把名字一行一行读进来,统计每个名字的字符种类.对名字排序以后找第一个字符种类最多的就行了.

```cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
}
struct Person {
	char name[30]; 
	int v;
} p[110];
int n;
bool used[30];
bool cmp(const Person &a, const Person &b) {
	return (strcmp(a.name, b.name)<0);
}

void gao(){
	scanf("%d",&n);
	getchar(); 
	int maxv = 0;
	for(int i=0;i<n;++i){
		gets(p[i].name);
		int l = strlen(p[i].name);
		memset(used,0,sizeof(used));
		int cnt = 0;
		for(int j=0;j<l;++j) {
			if(p[i].name[j]==' ')
			continue;
			if(!used[p[i].name[j]-'A']) {
				++cnt;
				used[p[i].name[j]-'A'] = 1;
			}
		}
		p[i].v = cnt;
		maxv = max(maxv,p[i].v);
	}
	sort(p,p+n,cmp);
	for(int i=0;i<n;++i)
	if(p[i].v==maxv) {
		puts(p[i].name);
		break;
	}
}
int main()
{
	useFile("A-large");
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

### [B. Rain](https://code.google.com/codejam/contest/11274486/dashboard#s=p1)

给一块$R × C$的板子,有$R × C$个格子,每个格子有个高度$H[i][j]$,现在往里面灌水,问他能蓄多少水(有些格子四周都比他高,那这个格子就能蓄水).

这题其实就枚举每个格子,这个位置的蓄水高度等于从他为起点,板子外面为终点的所有路径中,经过的最高格子最小的那条路径经过的最高格子.恩有点绕口...先考虑一条路径,如果水从这条路径流出去,那水位的高度至少要比路径经过的所有格子都要高,也就是这条路径使得水位不会高于这个值.然后dfs每条路径,取最小的限制值,就能得到结果.

不过,这玩意其实等价于求每个格子到板外的最短路径,路径长度等于路径中最高点的大小.那只要把外面看成一个点,求外面到每个格子的单源最短路,把得到的值和原值减一下就行了.

另外还有并查集的做法,我不会...
我这里是Dijkstra的做法.

```cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
}
int v[10001];
int ans[10001];
bool used[10001];
vector<int> nodes[10001];
struct Node {
	int _cur;
	int _max;
	friend bool operator <(const Node &a,const Node &b){
		return a._max > b._max;
	}
};
int steps[4][2] = {
	-1,0,
	0,1,
	1,0,
	0,-1
};
void dij(int cur) {
	priority_queue <Node> qu;
	Node tmp;
	tmp._cur=cur;
	tmp._max=0;
	qu.push(tmp);
	while(!qu.empty()) {
		tmp = qu.top();
		qu.pop();
		cur = tmp._cur;
		if(used[cur])
			continue;
		used[cur] = 1;
		ans[cur] = tmp._max;
		
		for(int i=0; i<nodes[cur].size();++i) {
			int next = nodes[cur][i];
			int nextans = max(ans[cur], v[next]);
			if(!used[next]) {
				Node nextnode;
				nextnode._cur = next;
				nextnode._max = nextans;
				qu.push(nextnode);
			}
		}
	}
}

void gao(){
	int n,m;
	cin>>n>>m;
	int lastnode = n*m;
	for(int i=0;i<=n*m;++i)
	nodes[i].clear();
	memset(used,0,sizeof(used));
	for(int i=0;i<n;++i) {
		for(int j=0;j<m;++j) {
			int curnode = i*m+j;
			scanf("%d", &v[curnode]);
			for(int k=0;k<4;++k) {
				int tmpi = i+steps[k][0];
				int tmpj = j+steps[k][1];
				if(tmpi<0||tmpi>=n||tmpj<0||tmpj>=m) {
					nodes[lastnode].push_back(curnode);
				} else {
					int nextnode = tmpi*m+tmpj;
					nodes[curnode].push_back(nextnode);
				}
			}
		}
	}
	dij(lastnode);
	int sum = 0;
	for(int i=0;i<n;++i){
		for(int j=0;j<m;++j) {
			int cur = i*m+j;
			sum += ans[cur]-v[cur];
		}
	}
	printf("%d\n",sum);
}
int main()
{
	useFile("B-large");
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

### [C. Jane's Flower Shop](https://code.google.com/codejam/contest/11274486/dashboard#s=p2)

给$M+1$个数,如$M = 3$, $4$个数$C\_0 = 10000$,$C\_1 = 3000$,$C\_3 = 4000$,$C\_4 = 5000$,求一个$r$满足:
$$-10000(r+1)^3 + 3000(r+1)^2 + 4000(r+1) + 5000 = 0$$
其中$-1 < r < 1$,$1 ≤ M ≤ 100$,$0 ≤ C\_i ≤ 1,000,000,000$,输入保证只有一个解.

天呐,智障题把我这个智障卡了好久.其实条件保证只有一个解,那就是说如果把式子左边看成函数,那函数在$-1 < r < 1$范围内和$0$只有一个交点.整个函数只有一个负项,就是第一项,也就是说函数只会先增加后减少,或者只减少.取$r = -1$函数一定不小于$0$.因此可以判定要有解的话$r=1$一定是不大于$0$的,然后瞎二分找取$0$的点就行了.

(我才不会说我之前根本没考虑这么多,因为想漏了所以这样写的)

```cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
}
int m;
int c[110];
double f(int m, double r) {
	double ans = 0;
	for(int i=0;i<=m;++i){
		ans*=(1+r);
		ans+=c[i];
	}
	return ans;
}
void gao(){
	scanf("%d",&m);
	for(int i=0;i<=m;++i)
		scanf("%d",&c[i]);
	c[0]=-c[0];
	double l=-1,r=1;
	double a0 = f(m,l);
	double a1 = f(m,r);
	if(a0>a1){
		for(int i=0;i<=m;++i)
		c[i]=-c[i];
	}
	while(r-l>=1e-10){
		double mid = (r+l)/2;
		double ans = f(m,mid);
		if(ans<=0)
			l = mid;
		else if(ans>0)
			r = mid;
	}
	if(f(m,l)==0)
	printf("%.9lf\n", l);
	else
	printf("%.9lf\n", r);
}
int main()
{
	useFile("C-large");
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

### [D. Clash Royale](https://code.google.com/codejam/contest/11274486/dashboard#s=p3)

有$N$张卡牌,每张卡牌有个当前等级$L[i]$和最高等级$K[i]$,卡牌$i$从等级$j$升级到$j+1$需要花费$C[i][j]$,卡牌$i$在等级$j$时攻击力为$A[i][j]$.现在给一个数$M$表示现有的金币可以用于升级,问升级后$8$张卡牌攻击力和最高是多少.

小数据只有$8$张牌,$M$也比较小,可以直接把卡牌的升级方案作为物品跑背包.

```cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
}
const int N = 15; 
const int K = 15;
const int M = 1010;
int m,n;
int k[N],l[N];
int a[N][K];
int c[N][K];
int dp[2][M];
void gao(){
	scanf("%d%d",&m,&n);
	for(int i=0;i<n;++i)
 	{
	 	scanf("%d%d",&k[i],&l[i]);
 		for(int j=1;j<=k[i];++j) {
		 	scanf("%d",&a[i][j]);
		 }
		 for(int j=2;j<=k[i];++j) {
 			scanf("%d",&c[i][j]);
 			c[i][j]+=c[i][j-1];
 		}
 	}
 	memset(dp,0,sizeof(dp));
 	for(int i=0;i<n;++i) {
	 	for(int j=l[i];j<=k[i];++j) {
	 		int cost = c[i][j] - c[i][l[i]];
	 		int cur = i%2;
	 		int bf = (i+1)%2;
	 		for(int kk=cost;kk<=m;++kk){
		 		dp[cur][kk] = max(dp[bf][kk-cost]+a[i][j],dp[cur][kk]);
		 		dp[cur][kk] = max(dp[cur][kk],kk==0?0:dp[cur][kk-1]);
		 	}
	 	}
	 }
	 printf("%d\n",dp[(n+1)%2][m]);
}
int main()
{
	useFile("D-small-attempt0");
	int t;
    scanf("%d",&t);
    for(int i=1;i<=t;++i) {
    	printf("Case #%d: ",i);
    	gao();
    }
    return 0;
}

```

大数据的话我并没有写出来.后来听了伟神的解法,感觉有点谜.
$N$最大为$12$,分成每边$6$张,两边先各自计算.枚举每种升级方案,得出取$2$到$6$张的时候有多少种升级方式可以选择.因为张数相同的时候肯定取花费小而攻击高的,所以可以按花费排序,去掉花费提高攻击却减小的情况, 维护一个单调性.
然后合并两边,使得一边取$i$张,一边取$8-i$张.枚举其中一边的每种升级方案,在另一边二分出满足花费限制攻击力最高的方案,保留最大值.
我觉得这个复杂度有点谜,听起来也挺麻烦的.

在朱神的指点下, 我直接维护取$1$到$8$张时每种情况的单调性, 也能很快出结果, 复杂度谜.

```cpp
//省略头文件
typedef long long ll;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
}
const int N = 15; 
const int K = 15;
ll m,n;
ll k[N],l[N];
ll a[N][K];
ll c[N][K];
struct Node {
	ll c,a;
	bool operator< (const Node& x) const {
		if(c==x.c)
			return a>x.a;
		return c<x.c; 
	}
};
vector<Node> collc[10];
void gao(){
	scanf("%lld%lld",&m,&n);
	for(int i=0;i<n;++i)
 	{
	 	scanf("%lld%lld",&k[i],&l[i]);
 		for(int j=1;j<=k[i];++j) {
		 	scanf("%lld",&a[i][j]);
		 }
		 for(int j=2;j<=k[i];++j) {
 			scanf("%lld",&c[i][j]);
 			c[i][j]+=c[i][j-1];
 		}
 	}
 	for(int i=0;i<=8;++i)
 		collc[i].clear();
	Node tmp;
	tmp.c = 0;
	tmp.a = 0;
	collc[0].push_back(tmp);
 	for(int i=0;i<n;++i) {
 		int lenar[10];
 		for(int kk=7;kk>=0;--kk) {
	 		lenar[kk] = collc[kk].size();
	 	}
	 	for(int j=l[i];j<=k[i];++j) {
	 		int cost = c[i][j] - c[i][l[i]];
	 		for(int kk=7;kk>=0;--kk) {
		 		for(int e=0;e<lenar[kk];++e) {
		 			tmp.c = collc[kk][e].c + cost;
		 			tmp.a = collc[kk][e].a + a[i][j];
		 			if(tmp.c<=m)
		 				collc[kk+1].push_back(tmp);
		 		}
		 	}
	 	}
	 	for(int kk=7;kk>=0;--kk) {
	 		sort(collc[kk+1].begin(),collc[kk+1].end());
	 		vector<Node> vtmp;
	 		for(int e=0;e<collc[kk+1].size();++e) {
 			if(e==0||collc[kk+1][e].a>vtmp[vtmp.size()-1].a)
				vtmp.push_back(collc[kk+1][e]);
	 		}
	 		collc[kk+1].clear();
	 		for(int e=0;e<vtmp.size();++e) {
 				collc[kk+1].push_back(vtmp[e]);
	 		}
	 	}
	 			 
	 }
	 printf("%lld\n",collc[8][collc[8].size()-1].a);
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
