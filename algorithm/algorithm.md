## STL 

### vector

+ 构造

  ```c++
  // c++11 初始化器列表语法：
  std::vector<std::string> words1 {"the", "frogurt", "is", "also", "cursed"};
  std::cout << "words1: " << words1 << '\n';
  
  // words2 == words1
  std::vector<std::string> words2(words1.begin(), words1.end());
  std::cout << "words2: " << words2 << '\n';
  
  // words3 == words1
  std::vector<std::string> words3(words1);
  std::cout << "words3: " << words3 << '\n';
  
  // words4 为 {"Mo", "Mo", "Mo", "Mo", "Mo"}
  std::vector<std::string> words4(5, "Mo");
  std::cout << "words4: " << words4 << '\n';
  ```

+ 访问

  ```c++
  const_reference front() const;
  const_reference back() const;
  void push_back( T&& value );
  void pop_back();
  // std::vector<T,Allocator>::insert
  iterator insert( iterator pos, const T& value ); // 在 pos 前插入 value
  iterator insert( const_iterator pos, size_type count, const T& value ); // 插入 count 个
  template< class InputIt > 
  void insert( iterator pos, InputIt first, InputIt last); // 插入  [first, last) 的元素
  ```

### list

```c++
std::list<int> list = { 8,7,5,9,0,1,3,2,6,4 };
list.sort(); // 0 1 2 3 4 5 6 7 8 9
list.sort(std::greater<int>()); // 9 8 7 6 5 4 3 2 1 0
list1.merge(list2); //合并两个已排序链表，若连个list均为降序，则需加std::greater<int>()为第二个参数
void push_front( T&& value );
void pop_front();
```

### map

```c++
std::map<int, std::string> map;
map.insert(std::make_pair(3, "test3"));
map.insert(std::pair<int, std::string>(4, "test4"));
```

### priority_queue

```c++
template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type>
> class priority_queue;
// push pop top 

// priority_queue 是容器适配器，它提供常数时间的（默认）最大元素查找，对数代价的插入与释出。
// 此 cmp 会使得 top 为最小元素，注意写法为 lhs > rhs;
struct cmp {
    bool operator() (const int& lhs, const int& rhs) { 
        return lhs > rhs;
    }
};
```

### heap

```c++
#include <algorithm>
template< class RandomIt, class Compare >
void make_heap( RandomIt first, RandomIt last, Compare comp );
// 在范围 [first, last) 中构造最大堆。
template< class RandomIt, class Compare >
void push_heap( RandomIt first, RandomIt last, Compare comp );
// 插入位于位置 last-1 的元素到范围 [first, last-1) 所定义的最大堆中。
template< class RandomIt, class Compare >
void pop_heap( RandomIt first, RandomIt last, Compare comp );
// 交换在位置 first 的值和在位置 last-1 的值，并令子范围 [first, last-1) 变为最大堆。
```

### string

+ 构造

  ```c++
  std::string s(4, '=');
  std::string a = "0123456789abcdefghij";
  std::string sub1 = a.substr(10); // abcdefghij
  std::string sub2 = a.substr(5, 3); // 567
  // str	-	要搜索的 string
  // pos	-	开始搜索的位置
  size_type find( const basic_string& str, size_type pos = 0 ) const; 
  // 在 str 所指的字节字符串中寻找字节字符串 target 的首次出现。不比较空终止字符。
  const char* strstr( const char* str, const char* target );
  ```

+ `string`和数值的转换

  ```c++
  int myint1 = std::stoi(str1);
  /*
  stol
  stoll
  stoul
  stoull
  stof
  stod
  stold
  */
  #include <cstdio>
int sprintf(char *str, const char *format, ...); // 写入 str
  int sscanf(const char *str, const char *format, ...); // 从 str 读
  ```

### algorithm

```c++
std::vector<int> data = { 1, 1, 2, 3, 3, 3, 3, 4, 4, 4, 5, 5, 6 };
// 返回指向范围 [first, last) 中首个不小于 value 的元素的迭代器，或若找不到这种元素则返回 last 。
auto lower = std::lower_bound(data.begin(), data.end(), 4);
// 返回指向范围 [first, last) 中首个大于 value 的元素的迭代器，或若找不到这种元素则返回 last 。
auto upper = std::upper_bound(data.begin(), data.end(), 4);
// 检查等价于 value 的元素是否出现于范围 [first, last) 中。
template< class ForwardIt, class T >
bool binary_search( ForwardIt first, ForwardIt last, const T& value );
```

## 常用算法

+ `bool binary_search( ForwardIt first, ForwardIt last, const T& value );`检查是否存在

+ `ForwardIt lower_bound( ForwardIt first, ForwardIt last, const T& value );`
  
  + 返回指向范围 `[first, last)` 中首个*不小于*（即大于或等于） `value` 的元素的迭代器，或若找不到这种元素则返回 `last` 。
  
+ `ForwardIt upper_bound( ForwardIt first, ForwardIt last, const T& value );`
  
  + 返回指向范围 `[first, last)` 中首个*大于* `value` 的元素的迭代器，或若找不到这种元素则返回 `last` 。
  
+  `std::priority_queue<int, std::vector<int>, std::greater<int> > q2;`

+  `std::sort (numbers, numbers+5, std::greater<int>());`

+ 比较函数

  ```c
  struct cmp {
    bool operator()(const pair<int, ListNode*>& lhs, 
    const pair<int, ListNode*>& rhs) {
    	return lhs.first < rhs.first;
  	}
  };
  ```

### [KMP](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)

```c++
// s    :   A   B   C   D   A   B   D
// next :   -1  0   0   0  -1   0   2   0  最后一个 0 可以用来做下一次匹配

// eg. A B C D A B C D B D E
void kmp_make_next(string& pattern, vector<int>& next) {
    int len = pattern.size();
    next.resize(len+1); // need default to be 0
    int i = 0, j = -1;
    next[0] = -1;
    while(i < len) {
        if(j == -1 || pattern[i] == pattern[j]) {
            ++i, ++j;
            next[i] = pattern[i] == pattern[j] ? next[j] : j;
        }
        else {
            j = next[j];
        } 
    }
}

int kmp_find(string& s, string& p, vector<int>& next) {
    int i = 0, j = 0, cnt = 0;
    while(j < s.size()) {
        if(i == -1 || p[i] == s[j]) ++i, ++j;
        else i = next[i];
        if(i == p.size()) i = next[i], ++cnt;
        // if(i == p.size()) return j-p.size()+1; // 返回第一次出现的位置
    }
    return cnt; // 返回出现的次数
}
```

### lower_bound和upper_bound

```c++
// 返回第一个 >= 的元素, 找不到则返回 data.size()
int lower_bound(std::vector<int>& data, int k) {
        int left = 0, right = data.size();
        while(left < right) {
                int mid = (left + right)/2;
                if(data[mid] < k) {
                        left = mid+1;
                }
                else {
                        right = mid;
                }
        }
        return left;
}

// 返回第一个 > 的元素, 找不到则返回 data.size()
int upper_bound(std::vector<int>& data, int k) {
        int left = 0, right = data.size();
        while(left < right) {
                int mid = (left + right)/2;
                if(data[mid] <= k) {
                        left = mid+1;
                }
                else {
                        right = mid;
                }
        }
        return left;
}
```

### 排序

#### quick_sort

```c++
// start = 0, end = a.size()
void quick_sort(vector<int>& a, int start, int end) {
  if(start >= end-1) return;
  int left = start, right = end-1;
  int pivotal = a[start];
  while(left < right) {
    while(left < right && a[right] >= pivotal) --right;
    a[left] = a[right];
    while(left < right && a[left] <= pivotal) ++left;
    a[right] = a[left];
  }
  a[left] = pivotal;
  quick_sort(a, start, left);
  quick_sort(a, right+1, end);
}
```

#### merge_sort

```c++
void merge_sort_core(vector<int>&a, vector<int>&tmp, int start, int end) {
  if(start >= end-1) return;
  int mid = (start+end)>>1;
  merge_sort_core(a, tmp, start, mid);
  merge_sort_core(a, tmp, mid, end);
  tmp = a;
  int i = start, j = mid, k = start;
  while(i < mid && j < end) {
    if(tmp[i] < tmp[j]) a[k++] = tmp[i++];
    else a[k++] = tmp[j++];
  }
  while(i < mid) a[k++] = tmp[i++];
  while(j < end) a[k++] = tmp[j++];
}
// start = 0, end = a.size()
void merge_sort(vector<int>& a, int start, int end) {
  vector<int> tmp(a);
  merge_sort_core(a, tmp, start, end);
} 
```

### 二叉树非递归遍历

```c++
struct TreeNode {
  TreeNode(int x): left(nullptr), right(nullptr), val(x) { }
  TreeNode *left, *right;
  int val;
};
// 先序
void pre_order(TreeNode *root) {
  if(!root) return;
  stack<TreeNode*> st;
  st.push(root);
  TreeNode *p;
  while(!st.empty()) {
    p = st.top();
    st.pop();
    // visit(p);
    if(p->right) st.push(p->right);
    if(p->left) st.push(p->left);
  }
}
// 中序
void in_order(TreeNode *root) {
  if(!root) return;
  stack<TreeNode*> st;
  TreeNode *p = root;
  while(!st.empty() || p != nullptr) {
    if(p) {
      st.push(p);
      p = p->left;
    }
    else {
      p = st.top();
      st.pop();
      // visit(p);
      p = p->right;
    }
  }
}
// 后序
void post_order(TreeNode *root) {
  if(!root) return;
  stack<TreeNode*> st;
  TreeNode *p = root, *q;
  do {
    while(p) {
      st.push(p);
      p = p->left;
    }
    q = nullptr;
    while(!st.empty()) {
      p = st.top();
      st.pop();
            // 右孩子不存在或已被访问，访问之
      if(p->right == q) { 
        // visit(p);
        q = p;
      } else {
        // 当前节点不能访问，需二次进栈
        st.push(p);
        // 先处理右子树
        p = p->right;
        break;
      }
    }
  } while(!st.empty());
}
```

### 动态规划

+ 可以得到最优解
+ 整体问题的最优解依赖各子问题的最优解
+ 这些小问题之间还有相互重叠的更小的子问题

#### 背包

```c++
// 01背包
for(int i = 0; i < n; ++i)
  for(int v = V; v >= cost[i]; --v)
    dp[v] = max(dp[v], dp[v-cost[i]] + weight[i]);

// 完全背包
for(int i = 0; i < n; ++i)
  for(int v = cost[i]; v <= V; ++v)
    dp[v] = max(dp[v], dp[v-cost[i]] + weight[i]);

// 多重背包，每种物品 Mi 个，体积 Ci, 价值 Wi，背包容量为 V
void MultiplePack(vector<int>& dp, int C, int W, int M) {
    if (C * M ≥ V) {
    	CompletePack(dp, C, W);
    	return;
	}
    k = 1
    while(k < M) {
    	ZeroOnePack(dp, k*C, k*W);
    	M -= k;
    	k += k;
    }
    ZeroOnePack(dp, C*M, W*M);
}
// 二维费用背包, 假定付出两种费用 u 和 v
// F[i, v, u] 表示前 i 件物品付出两种费用 u 和 v 时可获得的最大价值
// F[i, v, u] = max{F[i-1, v, u], F[i-1, v-Ci, u-Di] + Wi};
```

#### 最长非降子序列（LIS）

```c++
// f[i] = max{f[j]} + 1 (a[i] > a[j] && i > j)
```

#### 最长公共子序列（LCS）

```c++
// f[i][j] 表示 a 序列的前 i 个元素与 b 序列的前 j 个元素中最长公共子序列的长度
// f[i][j] = max( f[i-1][j], f[i][j-1], f[i-1][j-1] + (a[i] == b[j]) )
```

#### 相关题目

+ `uva1025`

## 数据结构

### 并查集

```c++
// 递归版本
int Find(int x) {
	return (set[x].parent==x) ? x : (set[x].parent = Find(set[x].parent));
}
// 非递归版本
int Find(int x) {
	int y=x;
	while (set[y].parent != y) 
        y = set[y].parent;
    while (x!=y) {  // 路径压缩，即把途中经过的结点的父亲全部改成y。
        int temp = set[x].parent;
        set[x].parent = y;
        x = temp;
    }
    return y;
}
```

### 树状数组

```c++
// hdu 1166 模板题
#include <vector>
#include <iostream>
#include <sstream>
#include <string>

using namespace std;

class BIT {
public:
  BIT(int n) { 
    data_.resize(n+1);  // index : 1...n
    n_ = n;
  }
  
  void add(int x, int d) {
    while(x <= n_) {
      data_[x] += d;
      x += lowbit(x);
    }
  }

  int sum(int x) {
    int ret = 0;
    while(x > 0) {
      ret += data_[x];
      x -= lowbit(x);
    }
    return ret;
  }

  int lowbit(int x) {
    return x & -x;
  }

private:
  vector<int> data_;
  int n_;
};

int main() {
  int T;
  scanf("%d", &T);
  for(int tt = 1; tt <= T; ++tt) {
    printf("Case %d:\n", tt);
    int n, x;
    scanf("%d", &n);
    BIT bit(n);
    for(int i = 1; i <= n; ++i) {
      scanf("%d", &x);
      bit.add(i, x);
    }
    string s;
    while(cin >> s) {
      if(s[0] == 'E') {
        break;
      }
      int i, j;
      scanf("%d%d", &i, &j);
      if(s[0] == 'A') bit.add(i, j);
      else if(s[0] == 'S') bit.add(i, -j);
      else if(s[0] == 'Q') {
        printf("%d\n", bit.sum(j) - bit.sum(i-1));
      }
    }
  }
  return 0;
}
```

### 线段树

#### 点修改

```c++
// hdu1754 模板题
#include <vector>
#include <iostream>
#include <algorithm>
#include <climits>

using namespace std;

class IntervalTree {
public:
  IntervalTree(int n) {
    max_.clear();
    max_.resize(3*n, 0);
    n_ = n;
  }

  void update(int p, int v, int o, int l, int r) {
    int mid = (l+r)>>1;
    if(l == r) max_[o] = v;
    else {
      if(p <= mid) 
        update(p, v, o*2, l, mid);
      else 
        update(p, v, o*2+1, mid+1, r);
      max_[o] = max(max_[o*2], max_[o*2+1]);
    }
  }

  int query(int ql, int qr, int o, int l, int r) {
    int mid = (l+r)/2, ans = INT_MIN;
    if(ql <= l && r <= qr) 
      return max_[o];
    if(ql <= mid) 
      ans = max(ans, query(ql, qr, o*2, l, mid));
    if(mid < qr) 
      ans = max(ans, query(ql, qr, o*2+1, mid+1, r));
    return ans;
  }

private:
  vector<int> max_; // max element in [ql, qr]
  int n_;
};

int main() {
        int n, m;
        char s[10];
        while(scanf("%d %d", &n, &m) != EOF) {
          int x;
    IntervalTree tree(n);
    for(int i = 1; i <= n; ++i) {
      scanf("%d", &x);
      tree.update(i, x, 1, 1, n);
    }
    while(m--) {
      int i, j;
      scanf("%s%d%d", s, &i, &j);
      if(s[0] == 'U') tree.update(i, j, 1, 1, n);
      else if(s[0] == 'Q') {
        printf("%d\n", tree.query(i, j, 1, 1, n));
      }
    }
  }
  return 0;
}
```

#### 区间修改（add）

```c++
// hdu 4267 模板题, 只要求区间和，注意sum可鞥超过INT_MAX
#include <vector>
#include <iostream>
#include <algorithm>
#include <climits>
#include <cstdio>

using namespace std;

template<typename T>
class IntervalTree {
public:
  IntervalTree(int n) {
    max_.resize(3*n, 0);
    min_.resize(3*n, 0);
    sum_.resize(3*n, 0);
    add_.resize(3*n, 0);
    n_ = n;
    _sum = 0, _min = 0, _max = 0;
  }

  void update(int ql, int qr, T v) {
    update_core(ql, qr, v, 1, 1, n_); 
  }

  
  void query(int ql, int qr) {
    _sum = 0, _min = 0, _max = 0;
    query_core(ql, qr, 1, 1, n_, 0);
  }

  T get_min() { return _min; }
  T get_max() { return _max; }
  T get_sum() { return _sum; }

private:
  void maintain(int o, int l, int r) {
    int lc = o*2, rc = o*2 + 1;
    sum_[o] = min_[o] = max_[o] = 0; 
    if(r > l) {
      sum_[o] = sum_[lc] + sum_[rc];
      min_[o] = min_[lc] > min_[rc] ? min_[rc] : min_[lc];
      max_[o] = max_[lc] > max_[rc] ? max_[lc] : max_[rc];
    }
    min_[o] += add_[o];
    max_[o] += add_[o];
    sum_[o] += add_[o] * (r-l+1);
  }


  void update_core(int ql, int qr, T v, int o, int l, int r) {
    if(ql <= l && r <= qr) {
      add_[o] += v; 
    }
    else {
      int mid = (l+r)>>1;
      if(ql <= mid) update_core(ql, qr, v, o*2, l, mid);
      if(qr > mid) update_core(ql, qr, v, o*2+1, mid+1, r);
    }
    maintain(o, l, r);
  }

  void query_core(int ql, int qr, int o, int l, int r, T add) {
    if(ql <= l && r <= qr) {
      _sum += sum_[o] + add*(r-l+1);
      _min = _min > min_[o]+add ? min_[o]+add : _min;
      _max = _max > max_[o]+add ? _max : max_[o]+add;
    } 
    else {
      int mid = (l+r)>>1;
      if(ql <= mid) query_core(ql, qr, o*2, l, mid, add+add_[o]);
      if(qr > mid) query_core(ql, qr, o*2+1, mid+1, r, add+add_[o]);
    }
  }


private:
  vector<T> max_;
  vector<T> min_;
  vector<T> sum_;
  vector<T> add_;
  int n_;
  T _min, _max, _sum;
};

int main() {
        int n, m;
        char s[10];
        while(scanf("%d %d", &n, &m) != EOF) {
          int x;
    IntervalTree<long long> tree(n);
    for(int i = 1; i <= n; ++i) {
      scanf("%d", &x);
      tree.update(i, i, x);
    }
    while(m--) {
      int i, j;
      scanf("%s%d%d", s, &i, &j);
      if(s[0] == 'C') {
        scanf("%d", &x);
        tree.update(i, j, x);
      }
      else if(s[0] == 'Q') {
        tree.query(i, j);
        printf("%lld\n", tree.get_sum());
      }
    }
  }
  return 0;
}
```

#### 区间修改（set）

```c++
// uva 11992 模板题
#include <vector>
#include <iostream>
#include <algorithm>
#include <climits>
#include <cstdio>

using namespace std;

const int INF = 1<<30;

template<typename T>
class IntervalTree {
public:
  IntervalTree(int n) {
    max_.resize(3*n, 0);
    min_.resize(3*n, 0);
    sum_.resize(3*n, 0);
    add_.resize(3*n, 0);
    set_.resize(3*n, -1); // -1 means no set value
    n_ = n;
    _sum = 0, _min = 0, _max = 0;
  }

  void update(int ql, int qr, T v, bool is_set) {
    update_core(ql, qr, v, 1, 1, n_, is_set); 
  }
  
  void query(int ql, int qr) {
    _sum = 0, _min = INF, _max = -INF; // may need change by type of T
    query_core(ql, qr, 1, 1, n_, 0);
  }

  T get_min() { return _min; }
  T get_max() { return _max; }
  T get_sum() { return _sum; }

  void get_result(T& minv, T& maxv, T& sumv) {
    if(minv > _min) minv = _min; 
    if(maxv < _max) maxv = _max; 
    sumv += _sum;
  }

private:
  void maintain(int o, int l, int r) {
    int lc = o*2, rc = o*2 + 1;
    sum_[o] = min_[o] = max_[o] = 0; 
    if(r > l) {
      sum_[o] = sum_[lc] + sum_[rc];
      min_[o] = min_[lc] > min_[rc] ? min_[rc] : min_[lc];
      max_[o] = max_[lc] > max_[rc] ? max_[lc] : max_[rc];
    }
    if(set_[o] >= 0) { 
      min_[o] = max_[o] = set_[o]; 
      sum_[o] = set_[o] * (r-l+1); 
    }
    if(add_[o]) {
      min_[o] += add_[o];
      max_[o] += add_[o];
      sum_[o] += add_[o] * (r-l+1);
    }
  }

  void pushdown(int o) {
    int lc = o*2, rc = o*2 + 1; 
    if(set_[o] >= 0) {
      set_[lc] = set_[rc] = set_[o];  
      add_[lc] = add_[rc] = 0;
      set_[o] = -1;
    }
    if(add_[o]) {
      add_[lc] += add_[o]; 
      add_[rc] += add_[o]; 
      add_[o] = 0;
    }
  }

  void update_core(int ql, int qr, T v, int o, int l, int r, bool is_set) {
    if(ql <= l && r <= qr) {
      if(is_set) set_[o] = v, add_[o] = 0;
      else add_[o] += v; 
    }
    else {
      pushdown(o);
      int mid = (l+r)>>1;
      if(ql <= mid) update_core(ql, qr, v, o*2, l, mid, is_set);
      else maintain(o*2, l, mid);
      if(qr > mid) update_core(ql, qr, v, o*2+1, mid+1, r, is_set);
      else maintain(o*2+1, mid+1, r);
    }
    maintain(o, l, r);
  }

  void query_core(int ql, int qr, int o, int l, int r, T add) {
    if(set_[o] >= 0) {
      int v = set_[o] + add + add_[o];   
      _sum += v * (min(qr, r) - max(ql, l) + 1);
      if(_min > v) _min = v;
      if(_max < v) _max = v;
    }
    else if(ql <= l && r <= qr) {
      _sum += sum_[o] + add*(r-l+1);
      _min = _min > min_[o]+add ? min_[o]+add : _min;
      _max = _max > max_[o]+add ? _max : max_[o]+add;
    } 
    else {
      int mid = (l+r)>>1;
      if(ql <= mid) query_core(ql, qr, o*2, l, mid, add+add_[o]);
      if(qr > mid) query_core(ql, qr, o*2+1, mid+1, r, add+add_[o]);
    }
  }

private:
  vector<T> max_;
  vector<T> min_;
  vector<T> sum_;
  vector<T> add_;
  vector<T> set_;
  int n_;
  T _min, _max, _sum;
};

int main() {
	int r, c, m;
	int op, x1, y1, x2, y2, v;
	int t_sum, t_min, t_max;
	while(scanf("%d%d%d", &r, &c, &m) != EOF) {
    vector<IntervalTree<int> > tree(r+1, IntervalTree<int>(c));
    while(m--) {
      scanf("%d%d%d%d%d", &op, &x1, &y1, &x2, &y2);
      if(op < 3) {
        scanf("%d", &v);
        for(int i = x1; i <= x2; ++i) 
          tree[i].update(y1, y2, v, op == 2);
      } else {
        t_sum = 0, t_min = INF, t_max = -INF; 
        for(int i = x1; i <= x2; ++i) {
          tree[i].query(y1, y2);
          tree[i].get_result(t_min, t_max, t_sum);
        }
        printf("%d %d %d\n", t_sum, t_min, t_max);
      }
    }
  }
  return 0;
}
```

#### 其它题型

+ `poj 2528`：查询`1~n`中不同值的个数

```c++
#include <vector>
#include <iostream>
#include <algorithm>
#include <climits>
#include <cstdio>
#include <set>
#include <map>

using namespace std;

class IntervalTree {
public:
  IntervalTree(int n) {
    set_.resize(n<<3, -1);
    n_ = n;
  }

  void update(int ql, int qr, int v) {
    update_core(ql, qr, v, 1, 1, n_); 
  }

  int query_diffnums() {
    set<int> count;
    query_diffnums_core(1, 1, n_, count);
    return count.size();
  }

private:
  void query_diffnums_core(int o, int l, int r, set<int>& count) {
    if(set_[o] >= 0) {
      count.insert(set_[o]);
      return;
    }
    if(l < r) {
      int mid = (l+r)>>1;
      query_diffnums_core(o*2, l, mid, count);
      query_diffnums_core(o*2+1, mid+1, r, count);
    }
  }

  void pushdown(int o) {
    if(set_[o] >= 0) {
      int lc = o*2, rc = o*2+1;
      set_[lc] = set_[rc] = set_[o];
      set_[o] = -1;
    } 
  }

  void update_core(int ql, int qr, int v, int o, int l, int r) {
    if(ql <= l && r <= qr) set_[o] = v; 
    else {
      pushdown(o);
      int mid = (l+r)>>1;
      if(ql <= mid) update_core(ql, qr, v, o*2, l, mid);
      if(qr > mid) update_core(ql, qr, v, o*2+1, mid+1, r); 
    }
  }

private:
  vector<int> set_;
  int n_;
};

int main() {
  int T;
  scanf("%d", &T);
  while(T--) {
    int n, x, y;
    scanf("%d", &n);
    vector<pair<int, int> > intervals;
    map<int, int> pos_idx;
    for(int i = 0; i < n; ++i) {
      scanf("%d%d", &x, &y);
      intervals.push_back(make_pair(x, y));
      pos_idx[x] = 1, pos_idx[y] = 1;
    }
    x = 1;
    for(map<int, int>::iterator it = pos_idx.begin(); it != pos_idx.end(); ++it, ++x) {
      it->second = x;
    }
    IntervalTree tree(x); 
    for(int i = 0; i < n; ++i) {
      int x1 = pos_idx[intervals[i].first];
      int x2 = pos_idx[intervals[i].second];
      tree.update(x1, x2, i+1);
    }
    printf("%d\n", tree.query_diffnums());
  }
  return 0;
}
```

+ `hdu1698`：`set`操作，求`1~n`的`sum`。
+ `zoj1610`：统计每个值在多少个连续不交叉的区间。
+ `poj3264`：查询区间最大最小值的差值

## 数学

- [数列网站](http://oeis.org/)

### 数论

#### 素数

```c++
void gen_primes(vector<int>& primes, vector<int>& vis, int n) {
  int m = (int)sqrt(n+0.5);  // <cmath>
  vis.resize(n+1, 0);
  primes.clear();
  for(int i = 2; i <= m; ++i) if(!vis[i])
    for(int j = i*i; j <= n; j += i) vis[j] = 1;
  for(int i = 2; i <= n; ++i) 
    if(!vis[i]) primes.push_back(i);
}
```

+ `hdu2136`：每个素数在素数表中都有一个序号，设 `1` 的序号为`0`，则 `2` 的序号为 `1`，输出给定的数`n`的最大质因子的序号
+ `hdu1397`：给出一个`n`，找出两个素数`a`，`b`且`a+b=n`，问有多少对这样的素数
+ `hdu3792`：对于连续的素数`p(n+1)-p(n) = 2`成为一个素数对，问对于不超过`N`的素数有多少对素数对

#### gcd和lcm

```c++
int gcd(int a, int b) {
    if(a < 0) a = 0-a;
    return b == 0 ? a : gcd(b, a%b); 
}
// 非递归实现
int gcd(int a, int b) { 
    if(a < 0) a = 0-a;
    while(b) { 
        int t = a % b; a = b; b = t; 
    }
    return a; 
} 
int lcm(int a, int b) { return a/gcd(a, b)*b; }
```

+ `hiho1330`：每次都按照一种固定的顺序重排数组，那么最少经过几次重排之后数组会恢复初始的顺序
  + 即求每一位回到原点的最少步数的最小公倍数
+ `hdu1108`：求`lcm`模板题
+ `hdu2503`：求化简后的`a/b + c/d`的分数形式

#### [扩展欧几里德](https://blog.csdn.net/u014634338/article/details/40210435)

```c++
// 求解 ax + by = d;  d = gcd(a, b); 即找出 d, x, y
void extgcd(int a, int b, int& d, int& x, int& y) {
	if(!b) { d = a; x = 1; y = 0; }
	else { extgcd(b, a%b, d, y, x); y -= x*(a/b); }
}
```

#### 同余方程

```c++
int mod(int x, int n) { return (x%n+n)%n; } // 对负数进行调整
// 求 ax = 1 (mod n) 的解 x, 即 a 关于模 n 的逆
int inv(int a, int n) {
	int d, x, y;
	extgcd(a, n, d, x, y);
	return d == 1 ? mod(x, n) : -1;
}

// 输出线性模方程 ax = b (mod n) 的解 x 
void linear_mod_equation(int a, int b, int n, vector<int>& sol) {
	gcd(a, n, d, x, y);
	if(b%d) return; // no solution
	else {
        int last = x * (b/d) % n; // first solution
        sol.push_back(last); 
        for(i = 1; i < d; i++) { // 总共会有 d 个解
            last =  (last + n/d) % n;
            sol.push_back(last);
        }
    }
}

// 中国剩余定理，用于求解 x = a[i] (mod m[i]), 共 n 个这样的方程组成的方程组
int CRT(int n, vector<int>& a, vector<int>& m)
{
  int M = 1, w, d, x, y, ans = 0;
  for(int i = 0; i < n; i++) M *= m[i];
  for(int i = 0; i < n; i++){
    w = M / m[i];
    extgcd(m[i], w, d , x , y); 
    ans = (ans + y*w*a[i]) % M; // accumulate e*的和a
  }
  return (M + ans%M)%M; // adjust to [0, M-1]
}
```

+ `poj1006`：中国剩余定理解同余方程组模板题
+ `poj1061`：扩展欧几里德解线性方程

#### 快速幂

```c++
// 矩阵快速幂
class Matrix {
public:
  Matrix(int n, long long mod = 0) : n_(n), mod_(mod) {
    matrix_.resize(n, vector<long long>(n, 0));
  }

  void init(int *a) {
    int pos = 0;
    for(int i = 0; i < n_; ++i)
      for(int j = 0; j < n_; ++j) 
        matrix_[i][j] = a[pos++];
  }

  void mul(Matrix* rhs) {
    Matrix ans(n_, mod_);
    for(int i = 0; i < n_; ++i) 
      for(int j = 0; j < n_; ++j) 
        for(int k = 0; k < n_; ++k) {
          ans.matrix_[i][j] += matrix_[i][k] * rhs->matrix_[k][j];
          if(mod_ > 0) ans.matrix_[i][j] %= mod_;
        }
    matrix_ = ans.matrix_;
  }
  // a^x
  void pow(long long x) {
    Matrix ans(n_, mod_);
    for(int i = 0; i < n_; ++i) ans.matrix_[i][i] = 1;
    while(x) {
      if(x & 1) ans.mul(this);
      x >>= 1;
      mul(this); 
    }
    matrix_ = ans.matrix_;
  }
  
  long long allsum() {
    long long ans = 0;
    for(int i = 0; i < n_; ++i) 
      for(int j = 0; j < n_; ++j) {
        ans += matrix_[i][j];
        if(mod_ > 0) ans %= mod_;
      }
    return ans;
  }

  void printall() {
    printf("{\n");
    for(int i = 0; i < n_; ++i) 
      for(int j = 0; j < n_; ++j) 
        printf("%lld%c", matrix_[i][j], (j == n_-1 ? '\n' : ' '));
    printf("}\n");
  }

public:
  vector<vector<long long> > matrix_;
  int n_;
  long long mod_;
};

// 快速幂模 a^k % M
LL pow_mod(LL a, LL k, LL M){
    LL b = 1;
    while(k){
        if(k & 1) b = (a*b) % M;
        a = (a % M) * (a % M) % M;
        k /= 2;
    }
    return b;
}
```

#### 欧拉函数

```c++
// 欧拉函数：返回小于 n 且与 n 互素的整数的个数
int euler_phi(int n) {
  int m = (int)sqrt(n+0.5);
  int ans = n;
  for(int i = 2; i <= m; ++i) if(n%i == 0) {
    ans = (i-1) * ans/i;
    while(n%i == 0) n /= i;
  }
  if(n > 1) ans = (n-1) * ans/n;
  return ans;
}

// 1 ~ n 中所有数的欧拉 phi 函数值
void phi_table(int n, vector<int>& phi) {
  phi.resize(n+1, 0);
  phi[1] = 1;
  for(int i = 2; i <= n; ++i) if(!phi[i]) {
    for(int j = i; j <= n; j += i) {
      if(!phi[j]) phi[j] = j;
      phi[j] = (i-1) * phi[j]/i;
    }
  }
}
```

#### 素数测试和因式分解

```c++
// x*y mod n
LL mul_mod(LL x, LL y, LL n){
        LL T = floor(sqrt(n) + 0.5);
        LL t = T*T-n;
        LL a = x/T; LL b = x%T;
        LL c = y/T; LL d = y%T;
        LL e = a*c/T; LL f = a*c%T;
        LL v = ((a*d + b*c)%n + e*t) % n;
        LL g = v/T; LL h = v%T;
        LL ans = (((f + g)*t%n + b*d)%n + h*T)%n;
        while(ans < 0) ans += n;
        return ans;
}
// a^k mod p
LL pow_mod(LL a, LL n, LL p)
{
        LL ans = 1, d = a % p;
        do{
                if(n & 1) ans = mul_mod(ans, d, p);
                d = mul_mod(d, d, p);
        }while(n >>= 1);
        return ans;
}
//以a为基,n-1=x*2^t   a^(n-1) = 1(mod n)  验证n是不是合数
//一定是合数返回true,不一定返回false
//二次探测
bool check(LL a, LL n, LL x, LL t)
{
    LL ret = pow_mod(a, x, n);
    LL last = ret;
    for(int i=1; i<=t; i++)
    {
        ret = mul_mod(ret, ret, n);
        if(ret == 1 && last != 1 && last != n-1) return true;//合数
        last = ret;
    }
    if(ret != 1) return true;
    return false;
}

// miller_rabin()算法素数判定
//是素数返回true.(可能是伪素数，但概率极小)
//合数返回false;
bool miller_rabin(LL n, LL times = 15)
{
    if(n < 2) return false;
    if(n == 2 || n == 3 || n == 5 || n == 7) return true;
    if((n%2 == 0) || (n%3 == 0) || (n%5 == 0) || (n%7 == 0)) return false;//偶数
    LL x = n-1, t = 0;
    while((x&1) == 0) x >>= 1, ++t;
    for(int i = 0; i < times; ++i)
    {
        LL a = rand()%(n-1)+1;
        if(check(a, n, x, t)) return false;//合数
    }
    return true;
}

LL gcd(LL a, LL b) { 
    if(a < 0) return gcd(-a, b);
    return b == 0 ? a : gcd(b, a%b); 
}

LL pollard_rho(LL x, LL c)
{
    LL i = 1,k = 2;
    LL x0 = rand()%x;
    LL y = x0;
    while(1)
    {
        ++i;
        x0 = (mul_mod(x0,x0,x) + c) % x;
        LL d = gcd(y - x0, x);
        if(d != 1 && d != x) return d;
        if(y == x0) return x;
        if(i == k) y = x0, k += k;
    }
}

//对n进行素因子分解
void findfac(LL n, vector<LL>& ans)
{
    if(miller_rabin(n)) {
        ans.push_back(n);
        return;
    }
    LL p = n;
    while(p >= n) p = pollard_rho(p, rand()%(n-1)+1);
    findfac(p, ans);//递归调用
    findfac(n/p, ans);
}
```

+ poj1811：判断一个数是否是素数，若不是则返回最小的素因子

### [组合数学](https://oi-wiki.org/math/combination/)

#### 排列组合

+ 排列数有顺序要求，组合数没有顺序要求

  + 线排列：`P(n, r) = n(n-1)...(n-r+1) = n!/(n-r)!`
    + 当`n >= r >= 2`时有`P(n, r) = nP(n-1, r-1)`
  + 圆排列：`P(n, r)/r`即选出`r`个人围成一个圆的方案数
  + 组合：`C(n, r) = C(n-1, r) + C(n-1, r-1)`
  + 多重集的组合数
    + 每种元素无限个，选`r`个：`F(n, r) = C(n+r-1, r)`

+ 不相邻的排列数：`C(n-k+1, k)`即从`1~n-k+1`中选`k`个数以`Xi+i-1`映射到`1~n`中

+ 错位排列：`f(n) = (n-1)(f(n-1) + f(n-2))`即可得到长度为`n`的全错排列数

  + 前几项为：0，1，2，9，44，265
  + `hdu2048`：模板题，找全错排的概率
  + `hdu2049`：找`n`对中有`m`对错排的情况数

+ 递推式：

  ![image.png](http://ww1.sinaimg.cn/large/77451733ly1g6kzwlrlasj209y01wdfs.jpg)

  + 取 `n = m` 有

    ![image.png](http://ww1.sinaimg.cn/large/77451733ly1g6kzxhlcayj205201y0sl.jpg)

+ 对多项式`(a+b)^n`的展开求导或求积分可推得的公式

  ![image.png](http://ww1.sinaimg.cn/large/77451733ly1g6kzze9hfxj204z01ua9w.jpg)

  ![image.png](http://ww1.sinaimg.cn/large/77451733ly1g6kzzrhfckj206i021dfp.jpg)

  ![image.png](http://ww1.sinaimg.cn/large/77451733ly1g6l008x9jxj205601q0sl.jpg)

+ 根据组合意义可以证明的公式

  ![image.png](http://ww1.sinaimg.cn/large/77451733ly1g6l01k37xxj206l01ygli.jpg)

  ![image.png](http://ww1.sinaimg.cn/large/77451733ly1g6l01ut1boj206o021jra.jpg)

#### Catalan数列

+ `n`从`0`开始，前几项为：`1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796, 58786, 208012, 742900, 2674440, 9694845, 35357670, 129644790, 477638700, 1767263190`
+ 给一个凸多边形，用`n-3`条不相交的对角线把它分成`n-2`个三角形，求不同的方法数目
  + 边`V1Vn`在最终的剖分种一定会恰好属于某个三角形`V1VnVk`
  + 两条边将多边形划分为了一个`k`边形和一个`n−k+1`边形
  + 故`f(n) = f(2)f(n−1) + f(3)f(n−2) + ⋯ + f(n−1)f(2)`，其中`f(2)=f(3)=1`
  + 可求得`f(n+1) = f(n)*(4n-6)/n`
+ `n` 个 `0`， `n` 个`1`，排列`m`长度序列，满足任意前缀种`0`的个数不少于`1`的个数的序列的数量为：
  + `C(2n, n)/(n+1)`即为`Catalan`数列
  + 由`n`个`0`，`n`个`1`构成的，存在一个前缀`0`比`1`多的序列与由`n-1`个`0`与`n+1`个`1`排成的序列构成双射
+ 有`2n`个人排成一行进入剧场。入场费 `5` 元。其中只有`n`个人有一张 5 元钞票，另外 `n`个人只有 `10` 元钞票，剧院无其它钞票，问有多少中方法使得只要有 `10` 元的人买票，售票处就有 `5` 元的钞票找零
  + 将持`5`元者到达视作将`5`元入栈，持`10`元者到达视作使栈中某`5`元出栈
+ 一位大城市的律师在她住所以北`n`个街区和以东`n`个街区处工作。每天她走`2n`个街区去上班。如果他从不穿越（但可以碰到）从家到办公室的对角线，那么有多少条可能的道路？
+ 在圆上选择`2n`个点，将这些点成对连接起来使得所得到的`n`条线段不相交的方法数？
+ 一个栈（无穷大）的进栈序列为`1，2，3，...，n`有多少个不同的出栈序列?
  + 把`0`看成入栈操作，`1`看成出栈操作，即`0`的累计个数不小于1的排列有多少种
+ `n`个结点可够造多少个不同的二叉树？
  + 枚举左右子树大小
+ 有多少种不同的长度为 n 的括号序列？
+ `P = a1 * a2 * a3 * ... * an`，其中`ai`是矩阵。根据乘法结合律，不改变矩阵的相互顺序，只用括号表示成对的乘积，试问一共有几种括号化方案？

#### stirling数

+ 第一类斯特林数：`s(n, r) = (n-1)*s(n-1, r) + s(n-1, r-1),  n > r >= 1`
  + 把`n`个不同的球分成`r`个非空循环排列的方法数。考虑最后一个球，他可以单独构成一个非空循环排列，也可以插入到前面一个球的一侧
+ 第二类斯特林数：`s(n, r) = r*s(n-1, r) + s(n-1, r-1),  n > r >= 1`
  + 把`n`个不同的球放到`r`个盒子里，假设没有空盒。考虑最后一个球，若它和前面的某一个球放在同一个盒子里或单独放一个盒子

#### 鸽笼原理

+  `n+1` 个苹果，想要放到 `n` 个抽屉里，那么必然会有至少一个抽屉里有两个
+ `hdu1808`：
  + 已知`m`堆糖果的数量，挑选其中一堆或多堆(可全选)的和恰好能被整除`n`,如果存在输出任意种组合的序号，否则输出`no sweets`;
  + 首先得到这`n`个数的前缀和`sum`，其中`sum[0]=0`，这样我们就得到了`n+1`个数，将其模`m`后，因为`m <= n`，那么由鸽巢原理得这`n+1`个数中最少存在两个数相同，令其为`sum[i],sum[j]`，那么`sum[j]-sum[i]`必为`m`的倍数，即从原数列第`i+1`项到第`j`项之间的`j-i`个数的和必然为`m`的倍数

#### 容斥原理

+ `hdu1796`：给你`m`个正整数，问你从`1`到`n-1`有多少个数至少能被这`m`个数中的一个数整除
  + `m`小于`20`，直接用二进制的方式枚举选`m`个数当中的哪些，求这些数的最小公倍数，将所有奇数个数的结果数量减去所有偶数个数的结果数量即为最终结果数量

#### [母函数](https://www.cnblogs.com/linyujun/p/5207730.html)

+ 序列`{0,1，2，3，4，5...n}`的母函数就是`f(x)=0+x+2x^2+3x^3+4x^4+...+nx^n`
+ `hdu1028`：一个数字`n`能够拆成多少种数字的和，可以用母函数的思想或`DP`的思想来解

## 图论

### 最短路径

```c++
class Graph {
public:
  typedef pair<int, int> pii;

  Graph(int n): n_(n) { g_.resize(n+1); }

  void add_edge(int u, int v, int w) {
    g_[u].push_back(make_pair(v, w));
    g_[v].push_back(make_pair(u, w));
  }
  
  int dijkstra(int s, int t) {
    vector<int> visited(n_+1, 0);
    vector<int> dist(n_+1, INF);
    dist[s] = 0;
    priority_queue<pii, vector<pii>, greater<pii> > q;
    q.push(make_pair(dist[s], s));
    while(!q.empty()) {
      pii u = q.top();  q.pop();
      int x = u.second;
      if(u.first != dist[x]) continue;
      if(u.second == t) break;
      for(int i = 0; i < g_[x].size(); ++i) {
        int y = g_[x][i].first, w = g_[x][i].second;
        if(dist[y] > dist[x] + w) {
          dist[y] = dist[x] + w;
          q.push(make_pair(dist[y], y));
        }
      }
    }
    return dist[t];
  }

private:
  vector< vector<pii> > g_; 
  int n_;
};
```

+ `hiho1081`、`poj2387`：模板题

## 计算几何

```c++
class Point {
public:
  
  Point(double a = 0, double b = 0) : x(a), y(b) { }
  
  Point operator+(const Point& rhs) const { return Point(x + rhs.x, y + rhs.y); }
  Point operator-(const Point& rhs) const { return Point(x - rhs.x, y - rhs.y); }

  bool operator<(const Point& rhs) const { return x < rhs.x || (x == rhs.x && y < rhs.y); }
  
  double dot(const Point& rhs) const { return x*rhs.x + y*rhs.y; }
  double length() const { return sqrt(x*x + y*y); }
  double angle(const Point& rhs) const { return acos(dot(rhs)/length()/rhs.length()); }

public:
  double x, y;
};
```

## 资源网站

+ [oi-wiki](https://oi-wiki.org)

## others

1. [八数码的八境界](https://www.cnblogs.com/goodness/archive/2010/05/04/1727141.html)
2. [`lower_bound`和`upper_bound`](https://blog.csdn.net/chaiwenjun000/article/details/48345501)

## note

+ 有符号二进制右移：`10001010 >> 3 = 11110001`

+ 有符号二进制`10000000 = -128`,`10000000 - 1 = 01111111`即127

+ `1<<31`为`INT_MIN`，需写成`(long)1<<31`。`INT_MIN = -2147483648，INT_MAX = 2147483647`

+ `leetcode`上`int, long, long long` 分别为`4，8，8`字节。

+ 正数的原码，反码和补码都一样，负数的反码为符号位不变，其余取反，负数的补码为反码加1

  + [机器使用补码做运算](https://www.cnblogs.com/zhangziqiu/archive/2011/03/30/ComputerCode.html)

+ `double`类型不能直接比较`==`或`!=，应当自定义比较精度，如`0.000001`，因为二进制无法精确的表示小数

  ```c
  bool equal(double a, double b) {
      if ((a - b > -0.000001) && (a - b < 0.000001))
          return true;
      else
          return false;
  }
  // 直接比较出错的例子：16.1 * 100 + 0.9 * 100 == 17.0 * 100 返回值为false
  ```

+ **循环队列**

+ `gcc`函数：

  ```c
  int __builtin_popcount (unsigned int x); 
  int __builtin_popcountl (unsigned long x);
  int __builtin_popcountll (unsigned long long x);
  // Returns the number of 1-bits in x.
  ```

+ 对任何容器来说，`for(auto x : vector)`类似的用法都是把原来的原来的元素拷贝一遍，若用迭代器，则均为指针。

+ `std::getline`的返回值。

+ 如果要用`cin >> string`，那么最好打开`std::ios::sync_with_stdio(false);`，并且其余的输入输出全部用`cin`和`cout`，速度也很快