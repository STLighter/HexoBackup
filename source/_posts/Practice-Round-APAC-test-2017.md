---
title: Practice Round APAC test 2017 题解
date: 2016-06-29 20:11:49
categories: 题解
tags:
  - APAC
  - 算法
  - 题解
---

比GCJ简单多了,题目大都很暴力(虽然最后一题还是不大会)

---

### [A. Lazy Spelling Bee](https://code.google.com/codejam/contest/5254486/dashboard#s=p0)

给一个字符串$a$,问有多少种字符串$b$满足长度与$a$相同且每一位$b[i] = a[i] OR a[i-1] OR a[i+1]$,结果模$10^9 + 7$.Case数$T$不大于$100$,字符串长度$L$不大于$1000$.

直接枚举每一位,统计相邻的位上有几种字符,然后乘起来就行了.复杂度$O(TL)$.

``` cpp
//省略头文件
using namespace std;
typedef long long ll;
string s;
const ll M = 1000000007;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
} 
void gao(){
	cin>>s;	
	ll ans = 1;
	for(int i=0;i<s.length();++i) {
		int cnt = 0;
		bool used[30] = {0};
		for(int j=i-1;j<=i+1;++j) {
			if(j<0||j>=s.length())
				continue;
			if(!used[s[j]-'a']) {
				used[s[j]-'a'] = 1;
				++cnt;
			}
		}
		ans*=cnt;
		ans%=M;
	}
	printf("%lld\n",ans);
}
int main()
{
	//useFile("A-large-practice");
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

### [B. Robot Rock Band](https://code.google.com/codejam/contest/5254486/dashboard#s=p1)

给$4$个含有$N$个数的数组,和一个数$K$,从数组中各取一个数$a,b,c,d$,问有多少种情况满足$a$^$b$^$c$^$d=K$.其中$N$不大于$1000$,Case数$T$不大于$10$.

等价于找$a$^$b=c$^$d$^$k$的组数.枚举所有$a,b$的组合,扔到map里面计数.然后枚举$c,d$的组合,看有多少$c$^$d$^$k$在map里面,加一下就可以了.复杂度$O(T N^2 \log(N))$

``` cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
}
int n,k;
int a[4][1010];
void gao(){
	map<int,ll> m;
	scanf("%d%d",&n,&k);
	for(int i=0;i<4;++i)
	for(int j=0;j<n;++j)
	scanf("%d",&a[i][j]);
	
	for(int i=0;i<n;++i)
	for(int j=0;j<n;++j)
	++m[a[0][i]^a[1][j]];
	
	ll ans = 0;
	for(int i=0;i<n;++i)
	for(int j=0;j<n;++j)
	ans+=m[k^a[2][i]^a[3][j]];
	
	printf("%lld\n",ans);

}
int main()
{
	//useFile("B-large-practice");
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

### [C. Not So Random](https://code.google.com/codejam/contest/5254486/dashboard#s=p2)

给$6$个数$N,X,K,A,B$和$C$,表示有$N$个节点串联在一起,每个节点输入一个数$in$并且处理后输出一个数$out$.对于每个节点,有$A/100$的概率$out=in$&$K$,有$B/100$的概率$out=in$|$K$,有$C/100$的概率$out=in$^$K$.求对第一个节点输入$X$,最后一点节点输出的期望值.$N$不大于$10^5$,Case数$T$不大于$50$.

反正都是与或非$K$,每个节点输出可取的值的个数很有限,视为常数$a$,所以直接枚举每个点的输出暴力求,有点类似概率dp,只不过这里用map搞.复杂度$O(aTN\log(a))$,$a$是一个常数.

``` cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
}
int n,x,k,a,b,c;
void gao(){
	map<int,double> m[2];
	scanf("%d%d%d%d%d%d",&n,&x,&k,&a,&b,&c);
	m[0][x] = 1;
	for(int i=0;i<n;++i) {
		map<int,double> &from = m[i%2];
		map<int,double> &to = m[(i+1)%2]; 
		to.clear();
		for(map<int,double>::iterator it = from.begin();it!= from.end();++it) {
			//printf("%d\n",it->first);
			to[it->first&k] += it->second*a/100.0;
			//printf("%d %.10lf\n",it->first&k, it->second*a/100.0);
			to[it->first|k] += it->second*b/100.0;
			//printf("%d %.10lf\n",it->first|k, it->second*b/100.0);
			to[it->first^k] += it->second*c/100.0;
			//printf("%d %.10lf\n",it->first^k, it->second*c/100.0);
		}
	}
	map<int,double> &cur = m[n%2];
	double ans = 0;
	for(map<int,double>::iterator it = cur.begin();it!= cur.end();++it) {
		ans += it->first*it->second;
	}
	printf("%.10lf\n",ans);
}
int main()
{
	//useFile("C-large-practice");
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

### [D. Sums of Sums](https://code.google.com/codejam/contest/5254486/dashboard#s=p3)

给一个长度为$N$的数组,枚举左端点右端点求和,得到$N*(N+1)/2$个数,将这些数排序以后得到新数组.然后有$Q$个询问,对于每个询问有$l,r$,问对新数组$[l,r]$区间和是多少.Case数$T$不大于$10$,$Q$不大于$20$,$N$不大于$200000$.

根本不会,看数据范围应该是根据什么性质去二分,只会暴力的去做small(1 ≤ N ≤ 10^3).就枚举一下原数组,$O(N^2)$的生成新数组,然后排序求前缀和.对于每个询问减一下就行了.复杂度$O(N^2 \log(N)+Q)$

``` cpp
//省略头文件
using namespace std;
typedef long long ll;
void useFile(string f) {
	freopen((f+".in").c_str(),"r",stdin);
	freopen((f+".out").c_str(),"w",stdout);
}
int n,q,l;
int a[1010];
ll b[1000100];
void gao(){
	scanf("%d%d",&n,&q);
	for(int i=0;i<n;++i) {
		scanf("%d",&a[i]);
	}
	b[0] = 0;
	l=1;
	for(int i=0;i<n;++i) {
		ll sum = a[i];
		b[l++] = sum;
		for(int j=i+1;j<n;++j) {
			sum += a[j];
			b[l++] = sum;
		}
	}
	sort(b+1,b+l);
	ll sum = 0;
	for(int i=0;i<l;++i) {
		b[i]+=sum;
		sum=b[i];
	}
	int x,y;
	for(int i=0;i<q;++i) {
		scanf("%d%d",&x,&y);
		printf("%lld\n",b[y]-b[x-1]);
	}
}
int main()
{
	//useFile("D-small-practice");
	int t;
    scanf("%d",&t);
    for(int i=1;i<=t;++i) {
    	printf("Case #%d:\n",i);
    	gao();
    }
    return 0;
}

```