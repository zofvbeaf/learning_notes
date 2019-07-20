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