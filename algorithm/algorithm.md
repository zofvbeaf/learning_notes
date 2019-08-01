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
    int i = 0, j = 0;
    while(s[j]) {
        if(i == -1 || p[i] == s[j]) ++i, ++j;
        else i = next[i];
        if(i == p.size()) return j-p.size(); // return first match pos
    }
    return -1;
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
void quick_sort(vector<int>& a, int start, int end) {
    int left = start, right = end;
    int pivotal = a[start];
    while(left < right) {
        while(left < right && a[right] >= pivotal) {
            --right;
        }
        a[left] = a[right];
        while(left < right && a[left] < pivotal) {
            ++left;
        }
        a[right] = a[left];
    }
    a[left] = pivotal;
    if(left > start)
      quick_sort(a, start, left-1);
    if(right < end)
      quick_sort(a, right+1, end);
}
```

### 动态规划

+ 可以得到最优解。
+ 整体问题的最优解依赖各子问题的最优解。
+ 这些小问题之间还有相互重叠的更小的子问题。

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

+ `poj 2528`：查询1~n中不同值的个数

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