---
title: 'Codeforces Round #544 (Div. 3)简单题解'
categories: 题解
tags:
  - Codeforces
  - 算法
  - 题解
date: 2019-03-17 15:52:55
---

复健，时间有限题解比较简陋

---

### [A. Middle of the Contest](https://codeforces.com/contest/1133/problem/A)

将小时转成分钟，得到起止时间在一天中的分钟数，取平均值即可，复杂度`O(1)`。平均值转换会时间的时候注意前导0。

```cpp
#include<cstdio>

int main() {
  int h0, m0, h1, m1, ans;
  scanf("%d:%d", &h0, &m0);
  scanf("%d:%d", &h1, &m1);
  ans = (h0 * 60 + m0 + h1 * 60 + m1) / 2;
  printf("%02d:%02d\n", ans/60, ans%60);
}
```

---

### [B. Preparation for International Women's Day](https://codeforces.com/contest/1133/problem/B)

要加起来能被`k`整除, 只需要看模`k`的余数即可。余数为`i`的与余数为`k-i`的互补可以被`k`整除，通过计数看有多少对能互补。需要注意的是，为余数为`0`和`k/2`（k为偶数）时，只能同余数的互补，此时计数是偶数个时都能配对，奇数个时能配对的数量是计数 - 1。复杂度`O(n + k)`。

```cpp
#include<iostream>
#include<cmath>
using namespace std;

int cnt[100];

int main() {
  int n, k, x;
  cin >> n >> k;
  while(n--) {
    cin >> x;
    ++cnt[x%k];
  }
  int ans = 0;
  if(cnt[0]) {
    ans += cnt[0] & -2;
  }
  for (int i = 1; i <= k/2; ++i) {
    if(i == k - i) {
      ans += cnt[i] & -2;
    } else {
      ans += min(cnt[i], cnt[k-i]) * 2;
    }
  }
  cout << ans << endl;
  return 0;
}
```

---

### [C. Balanced Team](https://codeforces.com/contest/1133/problem/C)

单调队列，从小到大添加元素，保证队首和队尾差不超过5，超过了则出队，否则用当前队列大小更新最优解。如果元素`x < y`则`x`一定比`y`先入队，而且能与`x`共存的最小值`sx`和能与`y`共存的最小值`sy`有`sx <= sy`。使用单调队列，每次入队后进行出队操作，出队完成后队首就是能与入队元素共存的最小值，队列内的元素就是以入队元素为最大值时所有能存在的元素。复杂度`O(n)`

```cpp
#include<iostream>
#include<algorithm>
using namespace std;

int a[200000];
int main() {
  int n;
  cin >> n;
  for (int i = 0; i < n; ++i) {
    cin >> a[i];
  }
  sort(a, a + n);
  int ans = 0, start = 0, end = 0;
  while(end < n) {
    if(a[end] - a[start] <= 5) {
      ans = max(ans, end - start + 1);
      ++end;
      continue;
    }
    ++start;
  }
  cout << ans << endl;
  return 0;
}
```

---

### [D. Zero Quantity Maximization](https://codeforces.com/contest/1133/problem/D)

`d * a[i] + b[i] = 0`可得`d = - b[i] / a[i]`，统计每种`d`取值的个数，取最大即可。对于`a[i]`为`0`的情况需要特殊讨论，如果`b[i]`也为`0`则此时`d`可以取任意值；否则，`d`的取值为`0`。另外，为了避免浮点误差，不能直接统计`d`，而是要统计`<a, b>`这个配对；同时，为了归一化，需要将`a`与`b`同时除以他们的最大公约数，并保证`a`是正数。复杂度`O(nlog(n))`,`log`是因为用了`map`来计数。

```cpp
#include<iostream>
#include<map>
#include<cstdlib>

using namespace std;

int a[200000], b[200000];

int gcd(int x, int y) {
  if(x > y) return gcd(y, x);
  if(x == 0) return y;
  return gcd(y%x, x);
}

int main() {
  int n;
  cin >> n;
  for(int i = 0; i < n; ++i) {
    cin>>a[i];
  }
  for(int i = 0; i < n; ++i) {
    cin>>b[i];
  }
  int zeroBCnt = 0, zeroBothCnt = 0;
  map<pair<int, int>, int> hash;
  for(int i = 0; i < n; ++i) {
    if(a[i] == 0) {
      if(b[i] == 0) ++zeroBothCnt;
      continue;
    }
    if(b[i] == 0) {
      ++zeroBCnt;
    }
    int divisor = gcd(abs(a[i]), abs(b[i]));
    a[i] /= divisor;
    b[i] /= divisor;
    if(a[i] < 0) {
      a[i] = -a[i];
      b[i] = -b[i];
    }
    if(hash[make_pair(a[i], b[i])]) ++hash[make_pair(a[i], b[i])];
    else hash[make_pair(a[i], b[i])] = 1;
  }
  int ans = zeroBCnt;
  for(auto item: hash) {
    if(item.second > ans) ans = item.second;
  }
  cout << ans + zeroBothCnt << endl;
  return 0;
}

```

—---

### [E. K Balanced Teams](https://codeforces.com/contest/1133/problem/E)

与C题思路类似，先得到取每个元素为最大值，能共存的元素有哪些（排过序的数组保留首位指针即可），比如位置为`i`的元素最小可共存元素的位置是`maxStart[i]`，这样数组里面`maxStart[i]`到`i`都是可共存元素。问题就转成如何在里面选`k`个，让元素尽量多，这样dp即可。`dp[i][j]`表示前`i`个元素选`j`队的最优值，则如果选`maxStart[i]`到`i`，最优值为`dp[maxStart[i] - 1], j - 1] + i - maxStart[i] + 1`；如果不选，最优值为`dp[i - 1][j]`；两者取最优得到状态转移方程。另外由于`j`只会从`j - 1`转移，因此可以用滚动数组节约内存。复杂度`O(nk)`。


```cpp
#include<iostream>
#include<algorithm>
using namespace std;

int a[5010];
int maxStart[5010];
int dp[5010][2];
int main() {
  int n, k;
  cin >> n >> k;
  for(int i = 0; i < n; ++i) {
    cin>>a[i];
  }
  sort(a, a + n);
  int l = 0, r = 0;
  while(r < n) {
    if(a[r] - a[l] <= 5) {
      maxStart[r] = l;
      ++r;
      continue;
    }
    ++l;
  }
  for(int i = 1; i <= k; ++i) {
    for(int j = 0; j < n; ++j) {
      if(maxStart[j]) {
        dp[j][i%2] = max(dp[j - 1][i%2], dp[maxStart[j] - 1][(i-1)%2] + j - maxStart[j] + 1);
        continue;
      }
      dp[j][i%2] = max(dp[j - 1][i%2], j - maxStart[j] + 1);
    }
  }
  cout << dp[n - 1][k%2] << endl;
  return 0;
}
```

---

### [F1. Spanning Tree with Maximum Degree](https://codeforces.com/contest/1133/problem/F1)

直接找到度最大的节点bfs即可，复杂度`O(n + m)`。

```cpp
#include<iostream>
#include<queue>
#include<vector>

using namespace std;

vector<int> node[200010];
bool visited[200010];

void bfs(int start) {
  queue<int> qu;
  qu.push(start);
  visited[start] = true;
  while(qu.size()) {
    int cur = qu.front();
    qu.pop();
    for(auto next : node[cur]) {
      if(visited[next]) continue;
      visited[next] = true;
      qu.push(next);
      cout << cur + 1 << ' ' << next + 1 << endl;
    }
  }
}

int main() {
  int n, m, x, y, tmp, maxCnt = 0, maxNode = -1;
  cin >> n >> m;
  for(int i = 0; i < m; ++i) {
    cin >> x >> y;
    --x;
    --y;
    node[x].push_back(y);
    node[y].push_back(x);
    tmp = node[x].size() > node[y].size() ? x : y;
    if(node[tmp].size() > maxCnt) {
      maxCnt = node[tmp].size();
      maxNode = tmp;
    }
  }
  bfs(maxNode);
  return 0;
}
```

---

### [F2. Spanning Tree with One Fixed Degree](https://codeforces.com/contest/1133/problem/F2)


如果从`1`的一个分支出发能从另一个分支回到`1`，则这些分支划分为同一组，dfs即可得到这些分组。如果一组里面所有分支都被去掉了，则这组里面的节点就无法出现在树里面，因此至少要保留一个。dfs得到有多少这样的组，每组里面取一个分支，剩下还可以取则任意取。这些分支作为bfs的第一步，继续搜下去，按搜索顺序输出即可。非法的情况有：dfs时存在节点没有走到；需要的度数比组数少（此时至少有一组所有分支都被去掉）；需要的度数比`1`链接的分支多。复杂度`O(n + m)`。

```cpp
#include<iostream>
#include<vector>
#include<queue>
#include<cstring>

using namespace std;
bool visited[200010];
vector<int> node[200010];
vector<int> group[200010];
queue<int> qu;
int n, m, d, g;

void dfs(int cur, int parent) {
  visited[cur] = true;
  for(auto next: node[cur]) {
    if(visited[next]) {
      if(parent == -1) group[g - 1].push_back(next);
      continue;
    }
    if(parent == -1) {
      group[g++].push_back(next);
    }
    dfs(next, cur);
  }
}

void bfs() {
  while(qu.size()) {
    int cur = qu.front();
    qu.pop();
    for(auto next: node[cur]) {
      if(!visited[next]) {
        visited[next] = true;
        cout << cur + 1 << ' ' << next + 1 << endl;
        qu.push(next);
      }
    }
  }
}

int main() {
  int x, y;
  cin >> n >> m >> d;
  for(int i = 0; i < m; ++i) {
    cin >> x >> y;
    --x;
    --y;
    node[x].push_back(y);
    node[y].push_back(x);
  }
  memset(visited, 0, sizeof(visited));
  dfs(0, -1);
  for(int i = 0; i < n; ++i) {
    if(!visited[i]) {
      cout << "NO" << endl;
      return 0;
    }
  }
  if(g > d || node[0].size() < d) {
    cout << "NO" << endl;
    return 0;
  }
  cout << "YES" << endl;
  memset(visited, 0, sizeof(visited));
  visited[0] = true;
  d -= g;
  for(int i = 0; i < g; ++i) {
    cout<< '1' << ' ' << group[i][0] + 1 << endl;
    visited[group[i][0]] = true;
    qu.push(group[i][0]);
    for(int j = 1; d && j < group[i].size(); ++j) {
      cout<< '1' << ' ' << group[i][j] + 1 << endl;
      visited[group[i][j]] = true;
      qu.push(group[i][j]);
      --d;
    }
  }
  bfs();
  return 0;
}
```
