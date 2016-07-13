---
title: leetcode题解(1-10)
date: 2016-07-13 12:21:26
categories: 题解
tags:
  - leetcode
  - 算法
  - 题解
---

### [1. Two Sum](https://leetcode.com/problems/two-sum/)

第一想法是枚举每个数,然后看总数减这个数的差值在不在原数组中.直接用map存数,枚举的时候在map里面查,复杂度`Nlog(N)`.
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        map<int,int> m;
        vector<int> ret;
        for(int i=0;i<nums.size();++i) {
            m[nums[i]] = i+1;
        }
        for(int i=0;i<nums.size();++i) {
            int b = m[target - nums[i]];
            if(b&&b!=i+1) {
                ret.push_back(i);
                ret.push_back(b-1);
                return ret;
            }
        }
        return ret;
    }
};
```
其实后来发现可以合并两个循环.
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        map<int,int> m;
        vector<int> ret;
        for(int i=0;i<nums.size();++i) {
            int b = m[target - nums[i]];
            if(b) {
                ret.push_back(i);
                ret.push_back(b-1);
                return ret;
            }
            m[nums[i]] = i+1;
        }
        return ret;
    }
};
```
<!-- more -->

试过用Mod方法hash一下..结果反而慢了不少..难道是vector的锅?
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        const int M = 100007;
        vector<vector<int> > hash(M,vector<int>());
        vector<int> ret;
        for(int i=0;i<nums.size();++i) {
            hash[(nums[i]%M+M)%M].push_back(i);
        }
        for(int i=0;i<nums.size();++i) {
            int b = target - nums[i];
            int p = (b%M+M)%M;
            for(int j=0;j<hash[p].size();++j)
            if(i!=hash[p][j]&&b==nums[hash[p][j]]) {
                ret.push_back(i);
                ret.push_back(hash[p][j]);
                return ret;
            }
        }
        return ret;
    }
};
```

后来想着如何减少枚举的次数.其实可以根据大小来分类,省去一些组合.但是这些操作需要更改元素的位置,所以需要事先记录一下.
两个数相加的话需要一个数小于`target/2`一个数大于`target/2`,或者两个数都等于`target/2`.
因为能保证只有一个答案,所以我们可以先把等于的情况特判掉.
然后将大于`target/2`的分在一起,小于`target/2`的分在一起(用类似快排的方式),将少的一边放入map,枚举另外一边(类似启发式合并的思路).
```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        struct node {
            int v;
            int idx;
        };
        map<int,int> m;
        vector<int> ret;
        vector<node> n_nums;
        int half = target/2;
        for(int i=0;i<nums.size();++i) {
            if(nums[i]==half) {
                ret.push_back(i);
                if(ret.size()>1)
                    return ret;
            }
            node tmp;
            tmp.v = nums[i];
            tmp.idx = i;
            n_nums.push_back(tmp);
        }
        ret.clear();
        node tmp;
        tmp.v = half;
        tmp.idx = -1;
        n_nums.push_back(tmp);
        int i=0,j=n_nums.size()-1,p=j;
        while(i<j) {
            while(i<j&&n_nums[i].v<n_nums[j].v) ++i;
            if(i==j)
                break;
            swap(n_nums[i],n_nums[j]);
            --j;
            while(i<j&&n_nums[i].v<=n_nums[j].v) --j;
            if(i==j)
                break;
            swap(n_nums[i],n_nums[j]);
            ++i;
        }
        if(i*2<n_nums.size()) {
            i = 0;
            for(;i<j;++i)
                m[n_nums[i].v] = n_nums[i].idx+1;
            i = j + 1;
            j = n_nums.size();
        } else {
            i = i + 1;
            j = n_nums.size();
            for(;i<j;++i)
                m[n_nums[i].v] = n_nums[i].idx+1;
            j = i - 1;
            i = 0;
        }
        for(;i<j;++i) {
            int t = target - n_nums[i].v;
            int idx = m[t];
            if(idx!=0) {
                ret.push_back(n_nums[i].idx);
                ret.push_back(idx-1);
                if(ret[0]>ret[1])
                    swap(ret[0],ret[1]);
                return ret;
            }
        }
        return ret;
    }
};
```
顺便一提,官方给的解法和我第一种差不多,但用java的hashmap(其实也就是第二种hash的思路).

---

### [2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)

简单的操作链表,记录一下之前有没有进位就行了,注意可能长度超过输入的两个链表中较长的那个哦(不过反正加法嘛,最多进位+1).复杂度`O(max(Length(l1),Length(l2))+1)`

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int v = l1->val + l2->val;
        ListNode *ret = new ListNode(v%10);
        ListNode *cur = ret;
        l1=l1->next;
        l2=l2->next;
        while(l1!=NULL||l2!=NULL) {
            v = (l1?l1->val:0) + (l2?l2->val:0) + v/10;
            cur->next = new ListNode(v%10);
            cur = cur->next;
            l1 = l1?l1->next:l1;
            l2 = l2?l2->next:l2;
        }
        if(v/10) {
            cur->next = new ListNode(1);
            cur = cur->next;
        }
        return ret;
    }
};
```

---

### [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)

要求是子串,也就是需要连续.我们可以把字符串看成一个队列.
如把`"abcabcbb"`前3个依次放进队列里面,得到`abc`.当放下一个字符`"a"`的时候,队列变成`abca`,有重复的字符了,这样需要把队首剔除,直到没有字符`"a"`为止.新的队列就变成`bca`了.
继续这个过程,我们就能得到每种符合要求的子串,保留最大的即可.
当然,一个一个出队很麻烦,不如直接记录字符出现的位置,直接更新子串的开始位置.复杂度$O(n)$.
顺便一说,英文字符的ASCII码只占0-127,所以m开128.
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
         int m[128] = {0};
         int ret = 0;
         int st = 0;
         for(int i=0;i<s.length();++i) {
             int p = m[s[i]];
             if(p) {
                 st = max(st,p);
             }
             ret = max(ret, i - st + 1);
             m[s[i]] = i+1;
         }
         return ret;
    }
};
```

---

### [4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)

如果两个数组组合成一个有序数组,实质上是要把整个数组平均分成两部分.这样分组还原到原数组的话,等价于把原先两个数组都分成两边,并且小的一边个数加起来应该是总数的一半.
这样我们划分一个数组,另一个数组就应该确定了划分的位置.我们只需要二分一个数组上的划分,然后让划分后的结果满足大小限制就行了.
复杂度`O(log(n)).`
```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int len1 = nums1.size();
        int len2 = nums2.size();
        int len = len1 + len2;
        int halflen = len/2;
        int l = max(0, halflen - len2);
        int r = min(halflen, len1);
        while(l<r) {
            int mid, mid2;
            mid = (l+r)/2;
            mid2 = halflen - mid;
            if(mid>0&&mid2<len2) {
                if(nums1[mid-1]>nums2[mid2]) {
                    r = mid - 1;
                    continue;
                }
            }
            if(mid<len1&&mid2>0) {
                if(nums1[mid]<nums2[mid2-1]) {
                    l = mid + 1;
                    continue;
                }
            }
            l = r = mid;
            break;
        }
        int right = min(l<len1?nums1[l]:0x7fffffff,halflen-l<len2?nums2[halflen-l]:0x7fffffff);
        int left = max(l>0?nums1[l-1]:-0x7fffffff,halflen-l>0?nums2[halflen-l-1]:-0x7fffffff);
        if(len&1)
            return 1.0*right;
        else
            return 1.0*(left + right)/2;
    }
};
```

---

### [5. Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/)

最长回文串,给了数据范围`1000`,就是在暗示用`O(n^2)`的做法么..就简单的枚举回文中心,然后枚举两侧求最长回文长度就行了,复杂度`O(n^2)`.
代码里面用插入额外字符的方式处理回文中心不是字符的情况(源自manacher).
```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        string ps = "#";
        string ret;
        int retl = 0;
        int p;
        int tmp;
        for(int i=0;i<s.length();++i) {
            ps.push_back(s[i]);
            ps.push_back('#');
        }
        for(int i=1;i<ps.length();++i) {
            tmp = 0;
            for(int j=i-1;j>=0;--j) {
                if(ps[j] == ps[2*i-j]) {
                    tmp += 1;
                } else {
                    break;
                }
            }
            if(retl<tmp) {
                p = i;
                retl = tmp;
            }
        }
        for(int i=p-retl;i<=p+retl;++i)
            if(ps[i]!='#')
                ret.push_back(ps[i]);
        return ret;
    }
};
```

附上manacher的解法,复杂度`O(n)`.(第一发写挂了..还是不熟练啊..欸)
```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        string str = "?#";
        string ret;
        for(int i=0;i<s.length();++i) {
            str.push_back(s[i]);
            str.push_back('#');
        }
        vector<int> dp(str.length(),0);
        int mx = 0;
        int id = 0;
        for(int i=1;i<str.length();++i) {
            if(i<mx)
                dp[i] = min(dp[id*2-i],mx-i);
            while(i-dp[i]>=0&&dp[i]+i<str.length()&&str[dp[i]+i]==str[i-dp[i]]) ++dp[i];
            if(dp[i]+i>mx) {
                mx = dp[i] + i;
                id = i;
            }
        }
        mx = 0;
        id = 0;
        for(int i=1;i<str.length();++i)
        if(dp[i]>mx) {
            mx = dp[i];
            id = i;
        }
        for(int i=id-mx+1;i<id+mx;++i)
            if(str[i]!='#')
                ret.push_back(str[i]);
        return ret;
    }
};
```

---

### [6. ZigZag Conversion](https://leetcode.com/problems/zigzag-conversion/)

找规律的题,只要找到新字符串字符在原字符串的位置之间的规律就行了.
比如样例中第一排字符在原字符串中间间隔`3`,第二排间隔`1`,第三排间隔`3`.
再推一下`numRows>3`的情况可知,第一排间隔是`dis = 2 * numRows - 3`,第二排是`dis - 2`和`2 - 1`交替出现, 第三排是`dis - 4`和`4 - 1`交替出现...
我代码里直接用步长,也就是间隔+1来计算的.

```cpp
class Solution {
public:
    string convert(string s, int numRows) {
        string ret;
        int step = max(1,2*(numRows-1));
        for(int i=0,j=0,k=step;i<numRows;++i,j+=2,k-=2) {
            int l = i;
            while(l<s.length()) {
                ret.push_back(s[l]);
                if(j==0||k==0) {
                    l+=step;
                } else {
                    l+=k;
                    if(l<s.length())
                        ret.push_back(s[l]);
                    l+=j;
                }
            }
        }
        return ret;
    }
};
```

---

### [7. Reverse Integer](https://leetcode.com/problems/reverse-integer/)

反转一个整数,一边模,另外一边乘和加就可以了,但是要注意可能会超int.直接用longlong存数然后判断一下就行了(ZZ题面正文都不说溢出要返回0).

```cpp
class Solution {
public:
    int reverse(int x) {
        long long ret = 0;
        long long maxint = 0x7fffffff;
        while(x!=0) {
            ret*=10;
            ret += x%10;
            x/=10;
        }
        if(ret>maxint)
            return 0;
        else if(ret<-maxint-1)
            return 0;
        return ret;
    }
};
```

---

### [8. String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/)

字符串转换成整数.题面又把需求藏起来..没啥说的,判判空字符,判判溢出之类的.有时间研究下atoi源码.
```cpp
class Solution {
public:
    int myAtoi(string s) {
        long long ret = 0;
        bool flag = 0;
        int i = 0;
        if(s.length()==0)
            return 0;
        while(i<s.length()&&s[i]==' ')
            ++i;
        if(s[i]=='-') {
            flag = 1;
            ++i;
        } else if(s[i]=='+') {
            flag = 0;
            ++i;
        }
        for(;i<s.length();++i) {
            if(s[i]>'9'||s[i]<'0') {
                break;
            }
            ret*=10;
            ret+= s[i] - '0';
            
            if(!flag && ret>INT_MAX)
                ret = INT_MAX;
            else if(flag && ret>1LL + INT_MAX)
                ret = 1LL + INT_MAX;
        }
        return flag?-ret:ret;
    }
};

```
---

### [9. Palindrome Number](https://leetcode.com/problems/palindrome-number/)

判断回文数,直接一位一位翻转,翻转到一半位数的时候判判相等就可以了.需要注意奇数位数和偶数位数.当然也可以都转换成字符串再判,复杂度稍微高点.
```cpp
class Solution {
public:
    bool isPalindrome(int x) {
        if(x<0)
            return false;
        int cmp = 0;
        while(cmp<=x){
            if(x==cmp||x/10==cmp) {
                return true;
            }
            cmp*=10;
            cmp+=x%10;
            x/=10;
            if(cmp==0)
                break;
        }
        return false;
    }
};
```

---

### [10. Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/)

明显的动态规划问题,`dp[i][j]`表示原串匹配到`i`,模式串匹配到`j`的时候是否匹配成功.
当`j`不为`*`的时候,当对应的单个字符匹配成功,`dp[i][j]`能从`dp[i-1][j-1]`转移.
当`j`为`*`的时候,如果匹配`0`次,`dp[i][j]`能从`dp[i][j-2]`转移.
当`j`为`*`的时候,如果是匹配第`i`次,且对应位置能再匹配,则能从匹配`i-1`次转移,即`dp[i][j]`能从`dp[i-1][j]`转移.
```cpp
class Solution {
public:
    bool isMatch(string s, string p) {
        int n = s.length();
        int m = p.length();
        vector<vector<int>> dp(n+1,vector<int>(m+1,0));
        dp[0][0]=1;
        for(int i=0;i<=n;++i)
        for(int j=1;j<=m;++j){
            if(p[j-1]=='*')
                dp[i][j] = dp[i][j-2]||(i>0&&dp[i-1][j]&&(p[j-2]=='.'||p[j-2]==s[i-1]));
            else
                dp[i][j] = i>0&&dp[i-1][j-1]&&(p[j-1]=='.'||(p[j-1]==s[i-1]));
        }
        return dp[n][m];
    }
};
```