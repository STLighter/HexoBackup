---
title: 用C++实现一些基础算法
date: 2016-07-23 16:09:54
categories: 算法
tags: 
  - 算法
---

## 排序

### 冒泡排序

```cpp
void bubble_sort(vector<int> &v) {
    int l = v.size();
    int tmp;
    bool flag;
    // 从右往左枚举i, v[i+1]到v[l-1]已经有序且是最终顺序, 每次循环将把v[0]到v[i]的最大值换给v[i]
    for(int i=l-1;i>0;--i) {
        // 从v[0]开始两两比较, 将大的数放到右侧, 最大的数自然被交换到了v[i]
        flag = false;   // 检测此次循环是否交换过, 如果没发生过交换则数组已经有序
        for(int j=0;j<i;++j) {
            // 两两比较
            if(v[j]>v[j+1]) {
                // 左侧较大则交换, 使得保持右侧较大
                tmp = v[j];
                v[j] = v[j+1];
                v[j+1] = tmp;
                flag = true;
            }
        }
        if(!flag)
            break;  //没有发生交换, 数组已经有序, 直接退出
    }
}
```
<!-- more -->

### 插入排序

和冒泡排序代码有点像, 但思想上完全不同, 反正我总是搞混.

```cpp
void insert_sort(vector<int> &v) {
	int l = v.size();
	int tmp;
	// 从左往右枚举i, v[0]到v[i-1]已经有序但不是最终顺序, 每次将v[i]放入其合适的位置
	for(int i=1;i<l;++i) {
        // 初始v[i] = v[j]
		for(int j=i;j>0;--j) {
		    // 将v[j]与前面的值比较
			if(v[j]<v[j-1]) {
			    // 如果v[j]比较小, 则与前面的值交换
				tmp = v[j];
				v[j] = v[j-1];
				v[j-1] = tmp;
			} else {
			    // 如果不是则已经保证了v[0]到v[i]有序
			    break;
			}
		}
	}
}
```

### 堆排序

先要建立小顶堆.

```cpp
class Heap {
private:
    // 内部用数组存储, 模拟二叉树, 根节点索引为0, 当节点索引为id时, 左儿子是id*2+1, 右儿子是id*2+2, 父节点是(id-1)/2
    vector<int> arr;
public:
    // 插入新的节点
    void push(int v) {
        int p = arr.size(), q, tmp;
        // 将新值加入数组尾部
        arr.push_back(v);

        // 从尾部开始维护小顶堆性质, 从加入位置开始检查, 如果父节点较大就交换并继续维护父节点. p是当前节点索引, q是父节点索引
        while(p!=0&&arr[p]<arr[(q = (p-1)/2)]) {
            tmp = arr[p];
            arr[p] = arr[q];
            arr[q] = tmp;
            p = q;
        }
    }
    int pop() {
        // 没有值可以pop();
        if(!arr.size()) {
            throw "error";
        }
        // 用ret存储根节点(堆顶)的值, 也就是要返回的值
        int ret = arr[0], p=0, q, tmp, l;
        // 将数组最后一个元素的值放到根节点, 并移除最后一个元素, 等价于交换后移除根节点的值
        arr[0] = arr[arr.size()-1];
        arr.pop_back();
        l = arr.size();

        // 从顶部开始维护堆的性质.
        while(1) {
            // 找到当前节点与两个儿子中较小值的角标
            if(p*2+1 < l && arr[p]>arr[p*2+1]) {
                q = p*2+1;
            } else {
                q = p;
            }
            if(p*2+2 < l && arr[q]>arr[p*2+2]) {
                q = p*2+2;
            }

            if(q!=p) {
                // 如果一个儿子的值更小, 则需要交换, 并继续维护儿子
                tmp = arr[p];
                arr[p] = arr[q];
                arr[q] = tmp;
                p = q;
            } else {
                // 当前节点最小, 则已经满足小顶堆性质
                break;
            }
        }
        return ret;
    }
    void clear() {
        arr.clear();
    }
};
```

通过将所有值插入堆, 并依次取出实现堆排序

```cpp
void heap_sort(vector<int> &v) {
    Heap heap;
    int l = v.size();
    // 插入堆, c++11语法
    for(auto i: v) {
        heap.push(i);
    }
    // 从堆中依次取出
    for(int i=0;i<l;++i) {
        v[i] = heap.pop();
    }
}
```

### 归并排序

```cpp
void merge_sort(vector<int> &v) {
    // 递归终止条件, 当少于一个元素, 数组有序, 直接返回
    if(v.size()<=1)
        return;
    int l = v.size();
    int half = l/2;
    // 将数组分为两段
    vector<int> left(v.begin(),v.begin() + half);
    vector<int> right(v.begin() + half, v.end());

    // 递归, 将两段分别排序
    merge_sort(left);
    merge_sort(right);

    // 合并两段, 将两段中左侧的值相互比较, 将较小的值放入原数组中
    for(int i=0,j=0,k=0;i<left.size()||j<right.size();) {
        // 左侧的数组段都放进了原数组, 则直接放右侧的数组
       if(i==left.size())
        v[k++] = right[j++];
        // 右侧的数组段都放进了原数组, 则直接放左侧的数组
       else if(j==right.size())
        v[k++] = left[i++];
        // 左侧数组的值小
       else if(left[i]<right[j])
        v[k++] = left[i++];
        // 右侧数组的值小
       else
        v[k++] = right[j++];
    }
}
```

### 快速排序

```cpp
void _qsort(vector<int> &v, int st, int end) {
    // 元素少于两个, 直接返回
    if(end - st < 2)
        return;
    // 将v[st]选为pivot, small_end表示一个索引, v[st+1]到v[small_end](包括v[small_end])都不大于v[st], 而v[small_end+1]之后都大于v[st]
    int small_end = st, pv = v[st], tmp;
    for(int i=st+1;i<end;++i) {
        if(v[i] <= pv) {
            // v[i]也小于pivot的值, 需要将v[i]放到small_end之后
            ++small_end;
            if(small_end!=i) {
                // small_end之后如果不是当前位置i, 则一定是一个大于pivot的值, 则直接和v[i]交换
                tmp = v[i];
                v[i] = v[small_end];
                v[small_end] = tmp;
            }
        }
    }
    if(small_end!=st) {
        // 如果不是所有值都大于pivot, 则pivot的正确位置不是st而是small_end.
        v[st] = v[small_end];
        v[small_end] = pv;
    }
    // 分别递归两侧
    _qsort(v, st, small_end);
    _qsort(v, small_end+1, end);
}
void quick_sort(vector<int> &v) {
    _qsort(v, 0, v.size());
}
```

---

## 数据结构

### 栈

```cpp
class st_stack {
private:
    // 最大容量
    int _size;
    // 实际大小
    int len;
    // 存储数据的数组
    int *arr;

    // 重新分配空间, 以便扩大容量
    void resize(int new_size) {
        int *tmparr = new int[new_size];
        memcpy(tmparr, arr, _size*sizeof(int));
        delete[] arr;
        arr = tmparr;
        _size = new_size;
    }
public:
    // 初始化栈的容量和大小
    st_stack() {
        len = 0;
        _size = 2;
        arr = new int[2];
    }
    // 释放arr占用的内存
    ~st_stack() {
        delete[] arr;
    }
    void push(int v) {
        if(len==_size) {
            // 如果容量已满, 扩容一倍
            resize(_size*2);
        }
        // 将数据加到数组尾部
        arr[len++] = v;
    }
    int pop() {
        // 从数组尾部取出数据
        if(len==0)
            throw "error";
        return arr[--len];
    }
    // 返回栈大小
    int size() {
        return len;
    }

    // 栈是否为空
    bool isEmpty() {
        return (len==0);
    }
};
```

### 队列

```cpp
class st_queue {
private:
    // 容量
    int _size;
    // 队列头
    int _start;
    // 队列尾
    int _end;
    // 队列长度
    int _len;
    // 存放队列的数组(循环队列)
    int *_arr;

    // 对数组扩容, 并将队列内容拷贝进新数组
    void resize(int new_size) {
        int *tmparr = new int[new_size];
        // 将原数组从队列头到数组结尾的位置复制进新数组的开始位置
        memcpy(tmparr, _arr + _start, sizeof(int)*(_size - _start));
        // 将原数组从数组头到队列尾的位置加进新数组
        memcpy(tmparr + _size - _start, _arr, sizeof(int)*_end);
        delete[] _arr;
        _arr = tmparr;
        _size = new_size;
        // 队列头和尾重新定位
        _start = 0;
        _end = _len;
    }

public:
    st_queue() {
        _start = 0;
        _end = 0;
        _len = 0;
        _size = 2;
        _arr = new int[2];
    }
    ~st_queue() {
        delete[] _arr;
    }
    void push(int v) {
        if(_len==_size) {
            // 队列已满则扩容
            resize(_size*2);
        }
        // 加到队列尾
        _arr[_end++] = v;
        // 更新队尾位置
        _end %= _size;
        // 更新队列长度
        ++_len;
    }
    int pop() {
        if(_len==0)
            throw "error";
        // 从队首取出
        int ret = _arr[_start];
        // 更新队首位置
        _start = (_start + 1) % _size;
        // 更新队列长度
        --_len;
        return ret;
    }
    bool isEmpty() {
        return (_len==0);
    }
    int size() {
        return _len;
    }
};

```

### 堆

参考前面堆排序部分

### 字典树(trie tree)

```cpp
class st_trie {
private:
    // 字典树节点
    struct TrieNode {
        // 表示是否为词尾
        bool isEnd;
        // 存储下一位的角标, -1表示暂时还没有节点. 这里假设字母只有26位小写字母
        int next[26];

        // 初始化
        TrieNode() {
            isEnd = false;
            memset(next,-1,sizeof(next));
        }
    };
    // 所有树节点, nodes[0]为根节点. 直接采用数组存储, 因为不删除所以数组不会有空位
    TrieNode *nodes;
    // 已用空间
    int _size;
    // 最大容量
    int _cap;
    // 扩容
    void resize(int new_cap) {
        TrieNode *tmp = new TrieNode[new_cap];
        memcpy(tmp, nodes, _cap*sizeof(TrieNode));
        delete[] nodes;
        nodes = tmp;
        _cap = new_cap;
    }
    // 插入字符串s, 当前搜到节点nodes[c], s[sp]是下一个字符
    void _insert(const string &s, int sp, int c) {
        if(sp == s.length()) {
            // 到达字符串结尾, 将标记变为true表示一个词结尾
            nodes[c].isEnd = true;
            return;
        }
        // 不是词尾则寻找表示下一个字符的节点
        int next = nodes[c].next[s[sp]-'a'];
        if(next == -1) {
            // 节点还不存在
            if(_size == _cap) // 如果容量已满则扩容一倍
                resize(_cap*2);
            // 分配新的节点
            next = nodes[c].next[s[sp]-'a'] = _size++;
        }
        // 递归调用, 插入下一个字符
        _insert(s, sp+1, next);
    }
    // 查询字符串, 参数意义与插入相同
    bool _find(const string &s, int sp, int c) {
        // 查询到词尾, 如果当前点标记为词尾则说明字典里有这个词, 否则没有
        if(sp == s.length())
            return nodes[c].isEnd;
        // 获取下一个字母节点
        int next = nodes[c].next[s[sp]-'a'];
        if(next == -1) // 下一个节点不存在, 则这个词一定不在字典里
            return false;
        // 查询下一个字母
        _find(s, sp+1, next);
    }
public:

    st_trie() {
        // 初始化nodes数组
        nodes = new TrieNode[_cap = 16];
        // _size为1因为有一个根节点
        _size = 1;
    }
    ~st_trie() {
        // 回收数组占用的内存
        delete[] nodes;
    }
    void insert(const string &s) {
        if(!s.length())
            return;
        _insert(s, 0, 0);
    }
    bool find(const string &s) {
        if(!s.length())
            return false;
        _find(s, 0, 0);
    }
};
```

### 哈希表

```cpp
class st_hashTable {
private:
    // 表的实际key个数
    const static unsigned int Mod = 10007;
    // 用vector存放实际元素
    vector<string> arr[Mod];

    // 计算字符串的hash值
    unsigned int getStringHash(const string &str) {
        unsigned int ret = 0;
        for(int i=0;i<str.length(); ++i) {
            // 一般使用的ASCII码只有0-127
            ret<<=7;
            ret+=str[i];
            ret%=Mod;
        }
        return ret;
    }

    // 根据hash值查询
    bool _find(unsigned int hash, const string &s) {
        // 枚举当前hash下的每个值, 如果有一个与要查询的s相同则返回true
        for(int i=0;i<arr[hash].size();++i)
            if(arr[hash][i]==s)
                return true;
        return false;
    }
public:
    st_hashTable() {
    }
    ~st_hashTable() {
    }

    // 查询字符串是否在hash表中
    bool find(const string &s) {
        // 计算hash值
        unsigned int hash = getStringHash(s);
        // 实际查询
        return _find(hash, s);
    }

    // 将字符串添加到hash表中
    void add(const string &s) {
        // 计算hash值
        unsigned int hash = getStringHash(s);
        if(!_find(hash, s)) {
            // 当前字符串不在hash表中则加入, 否则不需要重复加入
            arr[hash].push_back(s);
        }
    }
};
```