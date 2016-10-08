---
title: leetcode题解(11-20)
date: 2016-10-05 14:01:59
categories: 题解
tags:
  - leetcode
  - 算法
  - 题解
---

### [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/)

如果直接枚举任意两个位置取最大蓄水量, 整体复杂度`O(N^2)`, 将会超时.
考虑如果已知`height[i]`为短板, 则最优值应选不比`height[i]`小的最远处的板.
假设短板在右侧, 我们可以枚举每个位置作为短板, 在其左侧找最远的长板.
长板有单调性条件:
`如果左侧的两块板中更靠右的板高度却不更高, 则取更靠左的作为长板一定更优.`
因此我们可以维护一个单调队列, 从左往右加入, 高度单调递增, 距离单调递减, 在其中二分查找最远的长板.

综合来看, 只需要从左往右枚举每个高度, 对每个高度, 在单调队列中查找不比他矮且离他最远的那个高度, 计算其蓄水量, 然后尝试插入当前高度, 更新队列.

如果短板在左侧, 只需要从右往左维护队列即可.
复杂度`O(Nlog(N))`.

```cpp
class Solution {
public:
    int bs(int i, vector<int>& a, vector<int>& h) {
        int l = 0, r = a.size()-1, mid;
        if(h[a[l]] >= h[i])
            return a[0];
        if(h[a[r]] < h[i])
            return i;
        while(l+1 < r) {
            mid = (l + r)/2;
            if(h[a[mid]] < h[i])
                l = mid;
            else if(h[a[mid]] > h[i])
                r = mid;
            else
                return a[mid];
        }
        return a[r];
    }
    int maxArea(vector<int>& height) {
        vector<int> monotone;
        int l = height.size();
        int ret = 0;
        monotone.push_back(0);
        for(int i=1;i<l;++i) {
            ret = max(ret,height[i]*(i - bs(i, monotone, height)));
            if(height[i]>height[monotone[monotone.size()-1]])
                monotone.push_back(i);
        }
        
        monotone.clear();
        monotone.push_back(l-1);
        for(int i=l-2;i>=0;--i) {
            ret = max(ret,height[i]*(bs(i, monotone, height) - i));
            if(height[i]>height[monotone[monotone.size()-1]])
                monotone.push_back(i);
        }
        
        return ret;
    }
};
```
<!-- more -->

还有更优美的`O(N)`解法. 
我们先看距离最远的两个板. 如果要通过选择其他板得到更大的蓄水量, 只有可能先修改短板的选择, 因为距离一定会减小, 要试蓄水量增大必须使短板变高.
更具体一点说, 假设以`[l, r]`为边界的值为`v(l, r)`, 如果`h[l] < h[r]`, 那么有`v(l, r - i) < v(l, r)`, 所以在`[l, r]`中间取的话, 最优值要么是`v(l, r)`, 要么就不会以`l`为边界的某个值, 于是可以把查询范围缩小.
这样的贪心只需要对数组扫描一次, 复杂度为`O(N)`, 代码也很短.

```cpp
class Solution {
public:
    int maxArea(vector<int>& height) {
        int ret = 0;
        for(int l=0,r=height.size()-1;l<r;){
            ret = max(ret, (r - l) * min(height[l], height[r]));
            if(height[l]<height[r])
                ++l;
            else
                --r;
        }
        return ret;
    }
};
```

---

### [12. Integer to Roman](https://leetcode.com/problems/integer-to-roman/)

数字转罗马数字, 就是找规律瞎搞.
其实可以把每一个十进制位拿出来单独看. 个位一般由`I`, `V`, `X`来构成, 而十位则由`X`, `L`, `C`来构成等等, 每位的构成规律是相同的, 根据规律瞎搞就可以了.

```cpp
class Solution {
public:
    string gao(int n, char& one, char& five, char& ten) {
        string ret;
        char base,top;
        if(n>=5) {
            base = five;
            top = ten;
            n -= 5;
        } else {
            base = NULL;
            top = five;
        }
        if(n < 4) {
            if(base)
                ret.push_back(base);
            for(int i=0;i<n;++i)
                ret.push_back(one);
        } else {
            ret.push_back(one);
            ret.push_back(top);
        }
        return ret;
    }
    string intToRoman(int num) {
        char RC[] = {'I', 'V', 'X', 'L', 'C', 'D', 'M', '?', '?'};
        string ret;
        int base = 1000, x, p = 6;
        while(base) {
            x = num/base;
            num = num%base;
            ret += gao(x, RC[p], RC[p+1], RC[p+2]);
            p -= 2;
            base/=10;
        }
        return ret;
    }
};
```

---

### [13. Roman to Integer](https://leetcode.com/problems/roman-to-integer/)

罗马数字转整数...继续规律瞎搞. 其实对于紧挨着的罗马数字, 只要右边的比左边的大, 那左边的一定是减去. 然后就转换过来加减一下就行了.

```cpp
class Solution {
public:
    int trans(char c) {
        switch(c) {
            case 'I':
                return 1;
            case 'V':
                return 5;
            case 'X':
                return 10;
            case 'L':
                return 50;
            case 'C':
                return 100;
            case 'D':
                return 500;
            case 'M':
                return 1000;
        }
        return 0;
    }
    
    int romanToInt(string s) {
        int l = s.length(),
            bf = 0,
            v,
            ret = 0;
        for(int i=l-1;i>=0;--i) {
            v = trans(s[i]);
            if(v<bf)
                ret -= v;
            else
                ret += v;
            bf = v;
        }
        return ret;
    }
};
```

---

### [14. Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)

找字符串公共前缀, 维护一个公共前缀长度`l`, 然后对每个字符串都去和第一个字符串比较, 看是否最长公共前缀长度能让变得`l`更小(即最多比较`l`长度, 都相同也可不再继续比较, 因为不能使得`l`更小). 最后返回任意一个字符传前`l`个字符组成的字符串即可. 复杂度`O(L)`, `L`为所有字符串长度之和.

```cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if(!strs.size()) {
            return "";
        }
        int l = strs[0].length();
        for(int i=1;i<strs.size();++i) {
            int j=0;
            for(;j<l&&j<strs[i].length();++j) {
                if(strs[i-1][j]!=strs[i][j])
                    break;
            }
            l = j;
        }
        string ret = strs[0].substr(0, l);
        return ret;
    }
};
```

---

### [15. 3Sum](https://leetcode.com/problems/3sum/)

三个数相加为0, 就枚举两个数, 对于其和求负数, 找是否存在第三个数等于这个值.
实际写的时候, 需要考虑去重和不能重复使用的问题, 我是通过排序后判断来做的.
判断第三个数存在性的时候用`set`来做, 但是为了不重复使用数字, 需要保证在`set`中的数一定在枚举的两个数的后面. 为了去重, 让第一个数不能多次选择相同的数.
复杂度`O(N^2)`.
其他方法可以参考下面一题.

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> ret;
        set<int> st;
        int l = nums.size();
        if(l<3)
            return ret;
        sort(nums.begin(),nums.end());
        for(int i=0;i<l;++i) {
            if(i!=0&&nums[i]==nums[i-1])
                continue;
            st.clear();
            for(int j=l-1;j>i;--j) {
                int sum = nums[i]+nums[j];
                if(st.find(-sum)!=st.end()) {
                    vector<int> tmp;
                    tmp.push_back(nums[i]);
                    tmp.push_back(nums[j]);
                    tmp.push_back(-sum);
                    ret.push_back(tmp);
                    while(j>i+1&&nums[j-1]==nums[j])--j;
                }
                st.insert(nums[j]);
            }
        }
        return ret;
    }
};
```

---

### [16. 3Sum Closest](https://leetcode.com/problems/3sum-closest/)

与目标最接近的三个数和. 我们先考虑两个数求和的情况.
对于一个有序数组`A[]`, 数字从小到大排列, 长度为`len`且不小于`2`. 设`A[x] + A[y]`是最优解, 则对所有`j > y`有`A[x] + A[j] > target`, `i < x`有`A[i] + A[y] < target`. 因此, 我们设置初始`i = 0; j = len - 1`, 当`A[i] + A[j] > target`则减小`j`, 如果`A[i] + A[j] < target`则增大`i`, 这样一定可以枚举到最优解. 因为`i`单调递增, `j`单调递减, `i = x`或者`j = y`两者必有一个先成立; 当一个成立时`A[i] + A[j]`与`target`的大小关系就能确定了, 则未成立的一个必然会继续变化, 直到两者均成立, 即枚举到了最优解. 这样的枚举过程是`O(N)`(当然要先`O(NlogN)`排序).
三个数的和就枚举第一个数, 用上面的方法求得加上另外两个数之后最接近目标值的结果, 保留最接近的值即可. 整体复杂度`O(N^2)`.

```cpp
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        sort(nums.begin(),nums.end());
        int l,r;
        int dis = 0x7fffffff;
        int ans = 0;
        int sum;
        for(int i=0;i<nums.size()-2;++i) {
            l = i+1;
            r = nums.size()-1;
            while(l<r) {
                sum = nums[i]+nums[l]+nums[r];
                if(dis>abs(target - sum)) {
                    dis = abs(target - sum);
                    ans = sum;
                }
                if(sum>target)
                    --r;
                else if(sum<target)
                    ++l;
                else
                    return ans;
            }
        }
        return ans;
    }
};
```

---

### [17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

乱搞题, 每按一个数字就枚举每种字符, 加入到之前的数字得到的答案后面. 注意按第一个按钮时前面的答案是空, 因此要特判一下(放一个空字符串也行, 只是要记得返回前给去掉).

```cpp
class Solution {
private:
    void gao(vector<string> &ret, string &chars) {
        vector<string> tmp;
        for(int i=0;i<chars.length();++i) {
            string str = "";
            if(ret.size()==0) {
                tmp.push_back(str + chars[i]);
            } else {
                for(int j=0;j<ret.size();++j) {
                    str = ret[j];
                    tmp.push_back(str + chars[i]);
                }
            }
        }
        ret.clear();
        for(int i=0;i<tmp.size();++i) {
            ret.push_back(tmp[i]);
        }
    }
public:
    vector<string> letterCombinations(string digits) {
        vector<string> ret;
        string str;
        for(int i=0;i<digits.length();++i) {
            switch(digits[i]-'0') {
                case 2:
                    gao(ret, str = "abc");
                    break;
                case 3:
                    gao(ret, str = "def");
                    break;
                case 4:
                    gao(ret, str = "ghi");
                    break;
                case 5:
                    gao(ret, str = "jkl");
                    break;
                case 6:
                    gao(ret, str = "mno");
                    break;
                case 7:
                    gao(ret, str = "pqrs");
                    break;
                case 8:
                    gao(ret, str = "tuv");
                    break;
                case 9:
                    gao(ret, str = "wxyz");
                    break;
                default:
                    ret.clear();
                    return ret;
            }
        }
        return ret;
    }
};
```

---

### [18. 4Sum](https://leetcode.com/problems/4sum/)

参考`第16题`, 排序, 枚举两个数, 然后用首尾指针的方式得到另外两个数, 然后用`set`去重.

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>> ret;
        int l = nums.size();
        if(l<4)
            return ret;
        set<vector<int>> s;
        sort(nums.begin(), nums.end());
        
        int mx = nums[l-1] + nums[l-2];
        int x,y,z,base;
        pair<int,int> key;
        for(int i=1;i<l-2;++i) {
            for(int j=i-1;j>=0;--j) {
                base = nums[i] + nums[j];
                if(base + mx < target)
                    break;
                x = i+1;
                y = l-1;
                while(x<y) {
                    z = base + nums[x] + nums[y];
                    if(z < target) {
                        ++x;
                    } else if(z > target) {
                        --y;
                    } else {
                        vector<int> tmp;
                        tmp.push_back(nums[j]);
                        tmp.push_back(nums[i]);
                        tmp.push_back(nums[x]);
                        tmp.push_back(nums[y]);
                        s.insert(tmp);
                        ++x;
                        --y;
                    }
                }
            }
        }
        for(auto i = s.begin();i!=s.end();++i) {
            ret.push_back(*i);
        }
        return ret;
    }
};
```

---

### [19. Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

删除链表尾第`N`个, 等于要找链表尾第`N + 1`个以便删除. 一般直接找到尾计算长度, 然后转换成链表头计数就很好找了, 但是显然不够好. 可以先用一个指针从头往后找`N`个, 再用另一个指针指向头节点, 然后一同往后扫, 直到前者指向链表尾, 这样后者就找到了要删除的点. 只要根据需要调整一下细节, 就能保证找到表尾第`N + 1`个点, 然后删除其后一个节点, 当然要特判删除头结点的情况.

```cpp
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if(!head)
            return head;
        ListNode* tmp = head;
        while(n--) {
            tmp = tmp->next;
        }
        if(!tmp) {
            tmp = head->next;
            delete head;
            head = tmp;
            return head;
        }
        ListNode* f = head;
        while(tmp->next) {
            tmp = tmp->next;
            f = f->next;
        }
        tmp = f->next;
        f->next = tmp->next;
        delete tmp;
        return head;
    }
};
```

---

### [20. Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)

经典题目, 判断括号合法性. 只要用栈保存左括号, 遇到右括号时看栈顶括号是否匹配, 匹配则配对出栈. 当所有括号枚举完都匹配成功且栈为空时表示所有的括号都匹配上了(如果只有一种括号可以直接用左括号加一, 右括号减一来实现, 然而这里是多种括号).

```cpp
class Solution {
public:
    bool isValid(string s) {
        stack<char> st;
        int l = s.length();
        for(int i=0;i<l;++i) {
            switch(s[i]) {
                case '(':
                case '[':
                case '{':
                    st.push(s[i]);
                    break;
                case ')':
                    if(!st.empty()&&st.top()=='(') {
                        st.pop();
                    } else {
                        return false;
                    }
                    break;
                case ']':
                    if(!st.empty()&&st.top()=='[') {
                        st.pop();
                    } else {
                        return false;
                    }
                    break;
                case '}':
                    if(!st.empty()&&st.top()=='{') {
                        st.pop();
                    } else {
                        return false;
                    }
                    break;
            }
        }
        return st.empty();
    }
};
```
