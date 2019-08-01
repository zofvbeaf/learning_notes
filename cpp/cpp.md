## `关键字

### volatile

+ 起源：最   早出现于 19 世纪 70 年代，被用于处理 memory-mapeed I/O (MMIO)带来的问题。

  ```c
  unsigned int *p = GetMagicAddress(); // 可能不是内存地址而是I/O设备地址
  unsigned int a, b;
  a = *p; //（1）读I/O设备第一个int
  b = *p; //（2）读I/O设备第二个int
  *p = a; //（3）写入I/O设备第一个int
  *p = b; //（4）写入I/O设备第二个int
  // (1)和(2) 的乱序或直接将 b = a 会造成问题
  // (3)和(4) 可能被编译器优化消除掉，无意义的赋值
  ```

+ 作用：

  + 易变性：让编译器每次操作该变量时一定要从内存中真正取出，而不是使用已经存在寄存器中的值

  ```c
  a = f(c);  
  b = a + 1;  // 此时可能会直接拿 rax 寄存器中的值。gcc 4.1会，8.1不会，加volatile关键字则都不会
  ```

  + 不可优化：即使是无用的常量也不会被优化消除掉，加`-O2`也不会
  + 顺序性：`gcc`的优化仅保证一段程序的输出，在优化前后无变化。可能会改变前后两行变量的赋值顺序。
    + `volatile`变量与非`volatile`变量间的赋值顺序也可能被调换，JAVA不会。
    + `volatile`变量间的赋值顺序不会被调换
    + `volatile`仅保证编译器不会乱序，不保证`CPU`乱序

+ 使用场景：

  + 内嵌汇编操纵栈可能会导致出现编译无法识别的变量改变；
  + 更多的可能是多线程并发访问共享变量时，一个线程改变了变量的值，怎样让改变后的值对其它线程 visible；
  + 中断服务程序中修改的供其它程序检测的变量需要加volatile； 
  + 多任务环境下各任务间共享的标志应该加volatile；
  + 存储器映射的硬件寄存器通常也要加volatile说明，因为每次对它的读写都可能由不同意义；

### static

+ 作用：

  + **修饰全局变量**

    + **表示该符号仅在本编译单元内（一个`.o`文件）可见**，也就是说，在链接时，这个名字不参与链接。

    + 在.h中声明并定义`static`全局变量，每次某文件x去`include`这个`.h`文件都会在x中产生一份该变量，其修改不影响其他`include`了同一个.h文件的编译单元，因为都是独一无二的。

      ```c
      //common.h
      static unsigned long val = 0xabcd; //如果此处不加static关键字，则编译会出现多重定义错误
      //a.c
      #include"common.h"
      void modify() {
          val = 0xdeaddead;
      }
      //b.c
      #include"common.h"
      #include<stdio.h>
      void show() {
          printf("val is 0x%lx\n", val);
      }
      //main.c
      #include "common.h"
      extern void modify();
      extern void show();
      int main() {
          show();
          modify();
          show();
          return 0;
      }
      // 结果打印出来的都是 val is 0xabcd, a.c中的修改不影响b.c中的show()的val
      ```

    + 解决办法如下：

      ```c
      //common.h
      extern unsigned long val;
      //common.c
      unsigned long val = 0xabcd;
      ```

    + 修饰全局对象：全局对象的构造在`main()`之前，析构在`main()`之后。

      + 这些操作的完成是通过编译时把这些初始化函数调用放到一个section中，运行时先运行这个section中函数指针指向的函数。
      + 全局变量的初始化的顺序是不确定的，所以，如果一个全局变量的初始化依赖于另一个全局变量，这可能会导致很大的问题，因为被依赖的全局变量可能还没有被初始化。

    + 全局变量位于`.bss`段或者`.data`段，取决于它的初始是否全为0.总而言之，全局变量的内存地址在加载elf文件时就已经确定了，不需要通过什么`new/malloc`之类的来分配。

    + `static`全局变量与普通全局变量的初始化是一样的，只是它们仅编译单元内可见。

  + **修饰局部变量**

    + **类`static`成员变量**：等价于修饰全局`static`变量，只是在访问这个变量时，需要用类名作为前缀修饰。
      + 静态成员属于类作用域，但不属于类对象
      + 不能在构造函数中进行初始化，因为其生命期与普通的静态变量一样，程序运行时进行分配内存和初始化，程序结束时则被释放。
      + 静态变量成员需要在class{}（类定义体）内声明，类定义体外定义和初始化
      + 静态常量成员可以在类定义体中直接初始化
    + **函数`static`变量**：在第一次访问时才做初始化的，当然，它的内存是早就已经分配好的。static局部变量的初始化在`C++11后`是保证线程安全的（c++11前不保证），故可用来实现单例模式。

  + **修饰全局函数**

    + `static`全局函数仅当前编译单元内可见

  + **修饰类成员函数**

    + 其实就是普通的C函数，普通的成员函数是会传递this指针的，而static类成员函数不会，这也是为什么不能在static类成员函数中使用this指针的原因。

### const

+ 作用：**通知编译器该变量是不可修改的**。

+ const修饰的变量是占用内存的，这与#define定义常量是不同的。

  ```c++
  int i = 0;
  int * const p1 = &i;    //p1不能变
  const int ci = 42;
  const int * p2 = &ci;   //p2指向的值不能变
  const int * const p3 = p2;      //p3和p3指向的值都不能变
  const int & r = ci;  // 仅 const 引用可以引用 const 类型
  ```

+ const引用可以用不同类型的对象初始化，因为 const 引用实际是绑定到了一个临时变量上。

  ```c
  double x = 1.1;
  const int& y = x; // 合法，相当于 int tmp = x; const int& y = tmp; 当然x改变，y读到的值也会变
  int& z = x; // 错误！因为 int(x) 转换后的结果是一个右值，只能绑定到 const 左值引用上
  ```

+ 在函数中，可以修饰参数和返回值。
+ 在类中，可以修饰类的成员函数，则表明其不能修改类的成员变量。
+ 对于类的成员函数，有时候必须指定其返回值为 const 类型，以使得其返回值不为“左值”。例如 `operator+`等运算符

### extern

+ 作用：声明（而非定义）函数或全局变量的作用范围，其声明的函数和变量可以在本模块活其他模块中使用

> 也就是说 B 模块(编译单元)要是引用模块(编译单元) A 中定义的全局变量或函数时，它只要包含 A 模块的头文件即可,在编译阶段，模块 B 虽然找不到该函数或变量，但它不会报错，它会在连接时从模块 A 生成的目标代码中找到此函数。

+ `extern "C"`：它告知编译器这个函数不需要使用`name mangling`机制，而是使用C的那一套机制。

### explicit

+ 禁止隐式转换：避免出现比如把一个int隐式转换为一个可通过int构造的对象。

### inline与define

+ `define`在预编译时做替换，避免了意义模糊的数字出现，不需要为这些常量分配存储空间
+ `define`缺乏类型检查机制，`const`可弥补这个缺点
  + 并且其会在符号表中被看到，调试方便。
  + const 对象有明确的类型，可以保证类型安全性。
  + const 对象有作用域的限制。
+ `inline`函数在编译时展开，而宏是由预处理时进行展开
  + 内联函数会检查参数类型，宏定义不检查函数参数 ，所以内联函数更安全。
  + 宏不是函数，而inline函数是函数，用宏实现的函数功能有不可预知性。
  + 宏在定义时要小心处理宏参数，（一般情况是把参数用括弧括起来）。
  + 函数并不能完全替代宏，有些宏可以在当前作用域生成一些变量，函数做不到。
+ c++类定义体内的函数默认会`inline`
+ 函数体过大时，编译器不会`inline`

## 概念

### new、delete、new[]和delete[]

+ `new/delete`、`new[]/delete[]` 要配套使用

+ 当用`new[]`为数组申请空间时，如果是基本类型或者传统`trivial deconstructor`类类型，可以使用`delete`进行释放，但是如果不是这种情况，必须使用`delete[]`进行析构。如果用`delete`进行释放，虽然能够调用一次析构函数，但事实上会出现内存访问出错，因为并不是指向真实的从`malloc`申请

+ `delete`如何知道要释放的数据的长度：`ptmalloc`在用户使用的一小段内存的前后需要加上辅助元数据信息，如下图：

  ![image](http://www.2cto.com/uploadfile/Collfiles/20160311/20160311092525198.png)
  
+ 对象数组：如果用`delete`释放只会释放第一个对象，应用`delete[]`，但仍有如下问题，最好办法就是用`vector`

  ```c++
  void f(A * pa)
  {
      delete[] pa;
  }
  B * pa = new B[10];
  f(pa);
  // 因为在f函数看来，数组中存放的是对象A，它会以sizeof(A)作为偏移来定位每个对象A，而实际上，数组存放的对象B，这样一来，对象就定位错了，这样一来，析构自然就出错了。
  ```

### 虚函数

+ **虚析构**：为了能够正确的调用对象的析构函数，一般要求具有层次结构的顶级类定义其析构函数为虚函数。因为在delete一个抽象类指针时候，必须要通过虚函数找到真正的析构函数。

+ **纯虚析构**：抽象类中用到，必须提供其定义。因为析构和构造函数都有固定的调用链，析构是从子类到父类逐个调用。

  ```c++
  class Base
  {
  public:
     virtual ~Base()= 0
  };
  Base::~Base() { } 
  ```

+ **构造函数不能是虚函数**：因为调用构造函数时会设置好虚表，如果是虚构造函数那么虚表还没有设置好，相悖。

+ **构造函数中不要调虚函数**，因为构造顺序是从父类到子类，不会调到子类的虚函数实现。

+ **override**

  ```c++
   // 父类
   virtual Base* clone();
   // 子类
   virtual Derived* clone() override; // 可以通过
  ```

## 对象模型

### 空对象

+ 单独一个空对象，大小是1，要确保两个不一样的对象拥有不同的地址

+ 空对象作为父类，会有空基类优化，空基类的大小为0，或者说空基类与子类位于同一内存位置上。这个优化不仅仅针对第一个空基类。

  ```c
  class A
  {};
  
  class B
  {};
  
  class D
  {};
  
  class C:public A, public B,public D
  {
  public:
  };
  
  int main()
  {
          cout << sizeof(C) << endl;
  }
  // 仍然为 1
  ```

### POD对象

+ 总体表现的和C语言的struct一样。能够直接memcpy进行拷贝。
  + 没有虚表指针
  + 没有虚基类指针
  + 使用编译器生成的ctor/dtor/copy ctor/move ctor/copy assign/move assign
  + 它自己的non-static成员变量和父类也要满足这一点
+ 很多时候C++对POD都有优化，比如在new一个class array时，如果class有自定义的dtor，除了分配足够大小的空间外，还需要额外分配一个空间用来记录array的大小（这样，在析构的时候，才能知道array的整体大小，以便于从后往前析构）。但是，对于POD，则不需要。

### 继承和虚表

> 怎么找虚函数的指针?
>
> 怎么获得虚函数真正需要的this指针?

+ 见《c++资料整理.docx》

+ 单继承：

  ```c++
  class A {
  public:
  	virtual ~A() {}
      virtual void f() {}
  private:
      int x;
  };
  
  class B : public A {
  public:
  	virtual ~B() {}
      virtual void f() override {}
  private:
      int y;
  };
  
  class C :public B {
  public:
  	virtual ~C() {}
      virtual void f() override {}
      virtual void g() {}
  private:
      int z;
  };
  /**               +------------+
  C 的内存布局        |      0     |
  +---------+       |typeinfo_ptr|
  | vtblptr | --->  |      f     |
  |    x    |       |     dtor1  |
  |    y    |       |     dtor2  |
  |    z    |       |       g    |
  +---------+       +------------+
  dtor1是不调用operator delte的，第二个调用operator delete
  g的位置是放在dtor2后面的，因为每个virtual function的布局需要与父类保持一致。
  **/
  ```

+ non-virtual菱形继承

  ```c++
  class A
  {
  public:
      A():x(0xaaaaaaaa){}
  int x;
  };
  
  class B:public A
  {
  public:
      B():y(0xbbbbbbbb){}
  int y;
  };
  
  class C:public A
  {
  public:
      C():z(0xcccccccc){}
  int z;
  };
  
  class D:public B, public C
  {
  public:
      D():w(0xdddddddd){}
      int w;
  };
  /**    
  每个对象再加两个虚函数f,g
  D 的内存布局    
  +---------+           
  | vtblptr |  ---> 虚表：对应 D，与 B 复用同一个。D::f, D::g, D::~D, D::~D
  |    x    |            D 额外增加的函数也添加在这里
  |    y    | 
  | vtblptr |  ---> 虚表：对应 C，通过 non-virtual thunk 跳回
  |    x    |  
  |    z    |  
  |    w    |  
  +---------+ 
  **/
  ```

+ virtual菱形继承

  ```c++
  class A{...  int x};
  class B:virtual public A{...  int y};
  class C:virtual public A{...  int z};
  class D:public B, public C{...  int w};
  /**
  D 的布局
  +---------+           
  | vtblptr |  ---> 虚表：对应B和D，但不包含仅写在A里的函数，这些函数通过先进行指针转换为虚基类再调用
  |    y    | 
  | vtblptr |  ---> 虚表：对应 C， 通过 virtual thunk 跳回
  |    z    |  
  |    w    |  ---> D::w
  |    x    |  ---> 对应虚基类 A::x 填在最后
  +---------+ 
  **/
  ```

+ 指针转换

  + **转虚基类指针：运行时确定**，读Derive的vtblptr，拿到虚表中的to-vbase-offset，将Derive加上对应的offset
  + 转非虚基类指针：能够在编译期确定偏移，直接this减去偏移就行了。

### 成员函数指针

+ 对于非virtual函数，它就是一个普通的函数指针，指向一个地址。
+ 对于virtual函数，它是该函数到虚表的偏移加一。如果这个值为0，则代表函数指针确实指向了NULL。
+ 区分virtual和非virtual很简单，只需要看它的最后一位是否为1.

## STL

#### allocator

+ 老版本因为有内存池，所以内存申请速度要比新版本快一些。但是在老版本里面，有两个十分严重的问题：
  + 二级配置器中的 free list 虽然参考了伙伴系统中的数据结构，但是**没有实现伙伴系统中的合并功能**。也就是说两个 8 字节的块儿无法合并成 16 字节的块儿，即使 `free list` 里面的内存够用，也要再去一级配置器里面 `malloc` 一个 16 字节的块儿。它本身的目的是：用不合并小块的策略来提升速度，但是**大大降低了内存使用效率。**
  + 即使程序员显式调用容器的析构函数，比如 `map` 的 `clear()`， `malloc` 也不把这些内存归还给 `OS`，而是缓存到自己的内存池。也就是说，它的内存池只会调用 `malloc` 申请内存，但从来不用 `free` 释放内存，所以**只要程序不结束，STL 为容器分配的内存空间就没办法释放。**
+ 对于 g++-3.4.* 以后的版本，allocator 只是对 operator new 和 operator delete 进行了简单的封装。
  + 因为现在高性能的分配器越来越多，比如 tcmalloc / jemalloc，所以已经完全没有必要再在 glibc 上提供一个专门用来分配内存空间的内存池了，何况那个内存池自身就有一些很严重的问题。

#### iterator

> 在STL中，原生指针也是一种迭代器，除了原生指针以外，迭代器被分为五类。

+ `Input Iterator`： 此迭代器不允许修改所指的对象，即是只读的。支持`==、!=、++、*、->`等操作。
+ `Output Iterator`：允许算法在这种迭代器所形成的区间上进行只写操作。支持++、*等操作。
+ `Forward Iterator`：  允许算法在这种迭代器所形成的区间上进行读写操作，但只能单向移动，每次只能移动一步。支持`Input Iterator`和`Output Iterator`的所有操作。
+ `Bidirectional Iterator`： 允许算法在这种迭代器所形成的区间上进行读写操作，可双向移动，每次只能移动一步。支持`Forward Iterator`的所有操作，并另外支持`--`操作。
+ `Random Access Iterator`： 包含指针的所有操作，可进行随机访问，随意移动指定的步数。支持前面四种Iterator的所有操作，并另外支持 `it + n、it - n、it += n、 it -= n、it1 - it2` 和 `it[n]` 等操作。
+ `it++` 返回的是对象的拷贝，`++it`返回的是对象的引用，一般尽量使用 `++it` 替换 `it++` 。

#### [traits](https://blog.csdn.net/my_business/article/details/7891687)

+ 就是把一个模板类的特性抽出来，封装到另一个新构造的模板类里，用来做中间层。当函数，类或者一些封装的通用算法中的某些部分会因为数据类型不同而导致处理或逻辑不同时（而我们又不希望因为数据类型的差异而修改算法本身的封装时），traits会是一种很好的解决方案。

+ traits的作用就是把不同容器的iterator和内置的指针类型类型统一起来，使它们通过trait后，提供相同的类型接口。

  ```c++
  template <typename T>
  struct TraitsHelper {
       static const bool isPointer = false;
  };
  template <typename T>
  struct TraitsHelper<T*> {
       static const bool isPointer = true;
  };
  
  //iterator_traites
  template<class Iterator>
  stuct iterator_traits{
      typedef typename Iterator::iterator_category iterator_category;
      typedef typename Iterator::value_type value_type;
      typedef typename Iterator::difference_type difference_type;
      typedef typename Iterator::pointer pointer;
      typedef typename Iterator::reference reference;
  };
  ```

+ 迭代器对应的 traits 类为 iterator_traits。

#### [vector](https://www.kancloud.cn/digest/stl-sources/177267)

+ iterator : RandomAccessIterator
+ vector容器是占用一段连续线性空间，所以vector容器的迭代器就等价于原生态的指针；
+ 每当配置内存空间时，可能会发生数据移动，回收旧的内存空间，如果不断地重复这些操作会降低操作效率，所有vector容器在分配内存时，并不是用户数据占多少就分配多少，它会分配一些内存空间留着备用，即是用户可用空间。
+ 内部数据结构：三个迭代器
  + `start`
  + `finish`，`start`到此处为`size()`
  + `end_of_storage`，`start`到此处为`capacity()`

+ 容量扩大时增长为为`__old_size + max(__old_size, __n)`，即旧空间的两倍或旧空间加上要新增的容量大小。

#### string

+ 元素是连续存储的

+ 内部结构：

  + 原子类型的引用计数
  + `char*` 指针，指向连续存储空间的开头位置
  + `size` 值
  + `capacity` 值

+ 引用计数是用来完成写时拷贝的：string 之间拷贝时不是深拷贝，只拷贝了指针， 也就是共享同一个字符串内容，并且会有一个引用计数原子地加1。只有在内容被修改的时候，才真正分配了新的内存并 copy，并对原string的引用计数减一。

  + 这就存在一个问题，比如A和B共享同一段内存，在多线程环境下同时对A和B进行写操作，可能会有如下序列：A写操作，A拷贝内存，B写操作，B拷贝内存，A对引用计数减一，B对引用计数减一，加上初始的一次构造总共三次内存申请，如果使用全拷贝的string，只会发生两次内存申请。

  + 一些不经意操作可能导致意外的内存拷贝。

    ```c++
    string s1("test for copy");
    string s2(s1);
    cout << s2 << endl; // shared
    cout << s2[1] << endl; // 此处会重新申请并拷贝内存, 因为s2是非const的，调用的是非const的[]
    // 类似的操作的还有at(), begin(), end()等, 非const的string调用这些方法都会导致额外的内存申请和拷贝, 所以好的编程习惯应该是“在const的场景下使用const”。
    ```

#### list

+ iterator：BidirectionalIterator
+ 实现是循环双向链表。
+ 单独实现了`sort()`，使用类似归并排序的思想：栈上建立64个元素的数组，每个存一个`2^i`长的链表的链表头，逐个遍历原始list的所有元素，先在第0个链表，之后合并放到第1个链表，再从头开始，类似2048游戏的思路。
+ `std::forward_list`是单向链表，比双向链表存储开销小。

#### deque

```c++
iterator insert( iterator pos, const T& value ); //在pos前插入value, 返回被插入value的迭代器。
reference front(); // const_reference front() const;
reference back();  // const_reference back() const;
void push_back( const T& value );
void pop_back();
void push_front( const T& value );
void pop_front();
```

+ `iterator : RandomAccessIterator`

+ 分段连续线性空间，随时可以增加一段新的空间。`deque`允许于常数时间内对头尾端进行插入或删除元素。

+  `deque` 的每次分配 `node`（缓冲区）都会分配完整的 512 个字节，使用完之后会继续分配新的缓冲区。

+  deque 有 `size` 和 `max_size` 方法获取已使用的空间和已分配的空间。

+ 为了管理这些分段空间deque容器引入了一种中控器`map`，`map`是一块连续的空间，其中每个元素是指向缓冲区的指针，缓冲区才是`deque`存储数据的主体。

+ `deque`容器具有维护`map`和迭代器的功能，`deque`定义的两个迭代器分别是`start`和finish，分别指向第一个缓冲区的第一个元素和最后一个缓冲区的最后一个元素的下一个位置；

+ 当对`deque`的元素进行排序时，为了提高效率，首先把`deque`数据复制到`vector`，利用`vector`的排序算法(利用`STL`的`sort`算法)，排序之后再次复制回`deque`。

+  deque 典型地拥有较大的最小内存开销；只保有一个元素的 deque 必须分配其整个内部数组（例如 64 位 libstdc++ 上为对象大小 8 倍； 64 位 libc++ 上为对象大小 16 倍或 4096 字节的较大者）

+ 一个deque相当大，占据80个字节。

  ```c++
  template<typename T>
  class deque{
      ...
  private:
      typedef T** map_pointer;
      deque_iterator<T> start;//32B
      seque_iterator<T> finish;//32B
      map_pointer map;//指向中央控制器头部, 8B
      size_type map_size; //8B
  };
  ```

+ 迭代器：

  ```c++
  struct _Deque_iterator {
  	...
     //以下是迭代器设计的关键，访问容器的节点
    _Tp* _M_cur;//指向缓冲区当前的元素
    _Tp* _M_first;//指向缓冲区的头(起始地址)
    _Tp* _M_last;//指向缓冲区的尾(结束地址)
    _Map_pointer _M_node;//指向中控器的相应节点
    ...
  };
  ```

#### queue

+ 无迭代器。

+ 基于`deque`，只能在队列头部进行移除元素，只能在队列尾部新增元素，可以访问队列尾部和头部的元素，但是不能遍历容器。

  ```c++
  void push( const value_type& value );
  void pop();
  reference front(); // const_reference front() const; 无元素时返回0
  reference back();  // const_reference back() const;
  ```

#### stack

+ 无迭代器。

+ 默认基于`deque`，只能在栈顶对数据进行操作

  ```c++
  reference top(); // const_reference top() const;
  void push( const value_type& value );
  void pop();
  ```

#### priority_queue

+ 无迭代器。
+ 只能在队列尾部加入元素，在头部取出元素。
+ 常数时间的（默认）最大元素查找，对数代价的插入与释出。
+ 底层实现是一个 heap。
+ 重载比较函数
  + 内部元素为`struct`，直接重载其的operator<
  + `std::priority_queue<int, std::vector<int>, std::greater<int> >`
  + 仿函数作为cmp函数，即struct内部重载`operator()`

#### [heap](http://blog.csdn.net/chenhanzhun/article/details/21086973)

+ `heap` 并不是一种容器，而是一种算法，任何能够提供随机访问迭代器的容器都能支持 `heap` 的操作。`heap` 不需要遍历内容，没有迭代器。`heap` 一般是基于 `vector` 容器的操作。

+ 位置为i的完全二叉树，左孩子位置为`2 * i + 1`，右孩子位置为`2 * i + 2`

+ **最大堆**：是指根节点的关键字值是堆中的最大关键字值，且每个节点若有儿子节点，其关键字值都不小于其儿子节点的关键字值。

  ```c++
  #include <algorithm>
  
  //检查范围[first, last)中的元素是否为（默认）最大堆。
  template< class RandomIt, class Compare >
  bool is_heap( RandomIt first, RandomIt last, Compare comp ); 
  // 在范围 [first, last) 中构造（默认）最大堆。
  void make_heap(...);
  void push_heap(...);
  void pop_heap(...);
  void sort_heap(...);
  ```

#### pair

+ 是一个struct，提供了两个成员变量first和second，提供了构造函数和拷贝构造函数，同时提供了两个最基本的操作operator==和operator<重载，其他的操作符重载都是基于前面两种的变形。

#### set

```c++
template<
    class Key,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<Key>
> class set;

//  返回由指向被插入元素（或阻止插入的元素）的迭代器和若插入发生则设为 true 的 bool 值。
std::pair<iterator,bool> insert( const value_type& value );
size_type count( const Key& key ) const;
iterator find( const Key& key ); // const_iterator find( const Key& key ) const;
// 返回容器中所有拥有给定关键的元素范围。范围以二个迭代器定义，一个指向首个不小于 key 的元素，另一个指向首个大于 key 的元素。
std::pair<iterator,iterator> equal_range( const Key& key );
iterator lower_bound( const Key& key ); // 返回指向首个不小于 key 的元素的迭代器。
iterator upper_bound( const Key& key ); // 返回指向首个大于 key 的元素的迭代器。
```

+ iterator：BidirectionalIterator
+ 基于红黑树，容器键key和值value是相同的。且在容器里面的元素是根据元素的键值自动排序的。

#### map

+ `iterator：BidirectionalIterator`
  + 不能用类似`map.begin() + 3`的操作
+ 基于红黑树

#### multiset和multimap

+ 允许键值重复，底层插入用的是`rb-tree`的`insert_equal`而非`insert_unique`。

#### unordered_(multi)map和unordered_(multi)set

+ iterator：ForwardIterator
+ 底层采用`hash table`，默认采用开链法来解决冲突问题。

#### hash table

+ 在每个表格元素中维护一个链表, 然后在链表上执行元素的插入、搜寻、删除等操作，该表格中的每个元素被称为桶(bucket)。
+ 解决碰撞（colllision）：
  + 线性探测（linear probing）：
    + 插入：当遇到相同的值的时候，顺序往下一一寻找，直到找到第一个空位。
    + 删除：惰性(laze)删除，找到需要删除的值，标记删除，等重新整理（rehashing），不惰性删除而直接置空则空位后还可能有元素。
  + 二次探测（quadratic probing）：
    + 插入：当遇到相同值的时候，向后移动 i^2，即：1，4，9，...
    + 删除：标记删除
  + 开链法（separate chaining）：
    + 为每一个表格元素维护一个list，插入和删除操作都在hash后对应的list上进行
    + STL即采用此做法
+ 扩展方式：虽然开链法不要求表格大小为质数，但仍按照质数扩展。实际从11，23，53开始的28个质数，每个至少是前一个的两倍。
  + `totalElementNum > MaxLoadFactor * currentBkts`时扩展
  + 桶扩展的时候会停止服务，为了减少扩展次数，我们可以调用`reserve`
+ 源码中实现的是`FNV（Fowler-Noll-Vo）`哈希

#### 迭代器失效

> 在使用erase方法来删除元素时，需要注意一些问题。

+ `vector`的迭代器在内存重新分配时将失效，删除元素时，指向被删除元素以后的任何元素的迭代器都将失效。

+ `list`的`erase`获取下一个有效位置，set，map等以不连续的节点形式存储的容器类似

  ```c++
  List.erase( itList++);  // 或
  itList = List.erase( itList);
  ```

+ `vector`和`deque`只能用`itVec = Vec.erase(itVec);`，第二种不行，对于序列式容器(如`vector,``deque`)，删除当前的iterator会使后面所有元素的iterator都失效。这是因为vetor,deque使用了连续分配的内存，删除一个元素导致后面所有的元素会向前移动一个位置。还好erase方法可以返回下一个有效的iterator。

## c++11

### 六大函数

```c++
class Widget
{
        Widget(){} // 默认构造函数
        Widget(const Widget &){}  // 复制构造
        Widget(Widget && ){}      // 移动构造
        Widget & operator=(const Widget &){}  // 复制赋值
        Widget & operator=(Widget &&){} // 移动赋值
        ~Widget(){}
};
```

### 四种cast

#### dynamic_cast

+ 用于将一个父类对象的指针转换为子类对象的指针或引用。这是一种运行期的类型检查。
+ 要求被转换的对象必须有虚函数，否则编译会报错。
+ 对于不合法的转换，dynamic_cast会失败，对于指针的转换，如果失败，会返回0，而对于引用，由于引用不可能指向一个空的obj，因此，引用转换失败时，会抛出bad_cast异常。

#### static_cast

+ 相当于C语言的cast
+ 不做运行时的检查，不如dynamic_cast安全。
+ 用于静态转换，可用于任何转换，但它不能用于两个不相关的类型转换，比如不能转化struct类型到int，不能转化指针到double等。另外，它不能在转换中消除const、volatile和__unaligned属性。

#### const_cast

+ 编译时的指令，去掉const属性或者volatile属性。

+ `const_cast`一个确定是`const`的数据，是未定义行为。

  ```c++
  const char * ps = "helloworld";
  const_cast<char *>(ps)[0] = 'a'; // 运行时会 segment fault
  ```

+ `const int a`真实实现是通过一个符号映射表来实现，如果不对a进行取地址，那么a在栈上不占任何空间，一旦对a取地址，那么会在栈上分配一个空间存储a对应的100，修改b时相当于修改了栈上的空间，真正访问a时，会直接参考符号映射表得到100。这也能反映出编译器对常量的保护是做得比较到位的。

#### reinterpret_cast

+ 编译时的指令，重新解释这块内存的类型。

### RAII

> Resource Acquisition Is Initialization，内存获取即初始化。很多东西都可以用RAII来管理，不仅仅是内存，还有锁，比如lock_guard等等。

#### shared_ptr

+ `m_ptr`
+ `m_refcount`
+ `use_count`：为0时，会dispose掉对象，仅析构掉对象，并不回收空间。
+ `weak_count`：另外受`weak_ptr`影响。为0时，destroy对象，不仅析构对象，也回收内存。

#### weak_ptr

+ 只影响`weak_count`的引用计数。
+ 它在`lock`的时候返回一个`shared_ptr`，如果`weak_ptr`指向的对象的`use_count`已经为空，那么它的返回的`shared_ptr`指向`nullptr`。
+ 解决**循环引用**问题。

#### enable_shared_from_this

+ 继承`enable_shared_from_this<T>`后，调用`shared_from_this()`可返回指向T的`shared_ptr`。

#### unique_ptr

+ `unique_ptr`的类型信息与`shared_ptr`不一样，`shared_ptr`的`deleter`不算作类型的一部分，而`unique_ptr`的`deleter`算作类型的一部分。也就是说，关于同一指针的不同`deleter`的`unique_ptr`不是同一种类型。
+ `unique_ptr`的`deleter`是跟着对象本身的，而没有放在什么控制块里面。`unique_ptr`中对象的所有权通过右值引用的方式转移。`std::move()`
+ 如果`deleter`是一个对象（仿函数、lambda）且没有任何成员变量的话，`deleter`实际上不占任何内存空间。但是如果`deleter`是一个函数指针，那么它必定占用内存空间。

### RTTI

> run time type info，每个类型（类，整型，enum，指针（函数指针，成员函数指针，对象指针，成员变量指针））都有对应的type info，都能通过typeid关键字来获取它们的typeinfo。
>
> 拥有虚表指针的类，可以在它们的虚表中找到它们的type info，而且，dynamic_cast的实现正是依赖于这些type info，因此，只有拥有虚表指针的类才能使用dynamic_cast。

### 右值引用

> 凡是真正的存在内存当中，而不是寄存器当中的值就是左值，其余的都是右值。其实更通俗一点的说法就是：凡是取地址（&）操作可以成功的都是左值，其余都是右值。
>
> `f(int && x)` ，`x`本身是左值，但它是一个右值引用。
>
> 函数的返回值不一定是右值，如`operator[]`函数

+ `std::move`无条件地把它的参数转换成一个右值，而`std::forward`只在特定条件满足的情况下执行这个转换。

+ `std::move`和`std::forward`在运行期都没有做任何事情。

+ `std::forward`只有在它的参数绑定到一个右值上的时候，它才转换它的参数到一个右值。会保留参数的左右值类型。

  ```c++
  class Test {
      int * arr{nullptr};
  public:
      Test():arr(new int[5000]{1,2,3,4}) { 
      	cout << "default constructor" << endl;
      }
      Test(const Test & t) {
          cout << "copy constructor" << endl;
          if (arr == nullptr) arr = new int[5000];
          memcpy(arr, t.arr, 5000*sizeof(int));
      }
      Test(Test && t): arr(t.arr) {
          cout << "move constructor" << endl;
          t.arr = nullptr;
      }
      ~Test(){
          cout << "destructor" << endl;
          delete [] arr;
      }
  };
  
  Test createTest() {
      return Test();
  }
  
  int main() {
      Test t(createTest());
  }
  ```
  + 打印结果为（编译时加上`-fno-elide-constructors`，关闭返回值优化（RVO），否则只有第一行和最后一行）：

  ```
  default constructor
  move constructor
  destructor
  move constructor
  destructor
  destructor
  ```

+ 与`std::move()`相区别的是，`move()`会无条件的将一个参数转换成右值，而`forward()`则会保留参数的左右值类型。

  ```c++
  template <typename T>
  void func(T t) {
      cout << "in func " << endl;
  }
  
  template <typename T>
  void relay1(T&& t) {
      cout << "in relay" << endl;
      func(t);  
  }
  
  template <typename T>
  void relay2(T&& t) {
      cout << "in relay " << endl;
      func(std::forward<T>(t)); // 
  }
  
  int main() {
      relay1(Test());  // func 的参数 t 调用 copy constructor, 为左值
      relay2(Test());  // func 的参数 t 调用 move constructor, 为右值
     	Test t;
      relay1(t);  // func 的参数 t 调用 copy constructor, 为左值
      relay2(t);  // func 的参数 t 调用 copy constructor, 为左值
  }
  ```

+ 通用引用（universal reference）：必须满足`T&&`的形式，类型`T`必须是通过推断得到的。

  ```c++
  template <typename T>
  class TestClass {
  public:
    //这个T&&不是一个通用引用，因为当这个类初始化的时候这个T就已经被确定了，不需要推断
    void func(T&& t) {} 
  }
  // 普通函数模版参数，是通用引用
  template <typename T>
  void f(T&& param);
  // auto 声明，是通用引用
  auto && var = ...;
  
  void f(const string & s) // 可以过编译，但去掉const就不行，"hello world"是个右值
  {
      cout << s << endl;
  }
  f("hello world");
  ```

### 引用与指针

+ 初始化：引用在创建时必须初始化，引用到一个有效对象；而指针在定义时不必初始化，可以在定义后的任何地方重新赋值。
+ 指针可以是NULL，引用不行。
+ 引用貌似一个对象的小名，一旦初始化指向一个对象，就不能将其他对象重新赋值给该引用，这样引用和原对象的值都会被更改。
+ 引用的创建和销毁不会调用类的拷贝构造函数和析构函数。
+ 指针是对象，占据内存；引用不是，不一定占据内存。（考虑int a; int& r = a;编译器做的优化，以及引用作为形参时必须占据存储）

### 对象切割（Object slicing）

+ 当把一个派生类对象赋给一个基类对象时，会发生对象切割。

+ 多态的实现是通过指针和引用；而对象的转换只会造成对象切割，不能实现多态。

  ```c++
  class Base 
  { 
  protected: 
      int i; 
  public: 
      Base(int a)     { i = a; } 
      virtual void display() 
      { cout << "I am Base class object, i = " << i << endl; } 
  }; 
    
  class Derived : public Base 
  { 
      int j; 
  public: 
      Derived(int a, int b) : Base(a) { j = b; } 
      virtual void display() 
      { cout << "I am Derived class object, i = "
             << i << ", j = " << j << endl;  } 
  }; 
    
  // Global method, Base class object is passed by value 
  void somefunc (Base obj) 
  { 
      obj.display(); 
  } 
    
  int main() 
  { 
      Base b(33); 
      Derived d(45, 54); 
      somefunc(b); 
      somefunc(d);  // Object Slicing, the member j of d is sliced off 
      return 0; 
  } 
  // 输出：
  // I am Base class object, i = 33
  // I am Base class object, i = 45
  ```

### lambda表达式

+ `[ captures 捕获列表 ] ( params ) -> ret { body }`

+ lambda有两种默认捕获模式，按值捕获或者按引用捕获。默认情况下`[]`不捕获任何东西，`[&]`表示按引用捕获每个变量，`[=]`表示按值捕获。

  ```c++
  template<typename T1, typename T2>
  auto f(T1 a, T2 b)->decltype(a+b)
  {
    return a+b;
  }
  ```

+ 闭包`（closure）`：是由一个`lambda`创建的运行期对象。闭包持有捕获数据的副本或引用
+ 闭包类`（closure class）`：闭包类是用于实例化一个闭包对象的，编译器为每个`lambda`参数一个独一无二的闭包类，`lambda`表达式中的代码会成为闭包类成员函数的可执行指令。
+ `lambda`与仿函数是等价的，完全可以用仿函数来仿照`lambda`，只是很不方便。
+ 不能捕获全局变量，不能捕获`static`或线程局部存储变量。
+ 捕获的变量可能在`lambda`被调用时生命周期已经结束。
+ 不能直接捕获类的成员变量，只有捕获`this`指针，才能在`lambda`里面直接用成员变量。

### function

+ `bind`的返回值不是`function`，而是一个仿函数。这个仿函数可以作为`function`的构造函数的参数，来构造一个`function`。其实，不用`function`来接`bind`的结果，而用auto来接也是完全可以的。

+ 就算你拿到了某个`lambda`表达式的类型，也无法用这个类型来创建一个`lambda`对象，即下面的表达式是不能过编译的，原因在于`lambda`类型的构造函数被声明为`delete`的了。

  ```c++
  auto func = []{cout << "helloworld" << endl;};
  decltype(func) funcopy;
  funcopy();
  ```

### nullptr

```c++
const class nullptr_t
{
public:
    template<class T>
    inline operator T*() const
        { return 0; }

    template<class C, class T>
    inline operator T C::*() const
        { return 0; }
 
private:
    void operator&() const;
} nullptr = {};
void f(sometype1* a, sometype2* b); // f(p, nullptr) 会调这个
void f(sometype1* a, int b); // f(p, NULL) 会调这个
```

### std::decay和std::is_same

+ `std::decay`就是对一个类型进行退化处理，把各种引用啊什么的修饰去掉，把cosnt int&退化为int，这样就能通过`std::is_same`正确识别出加了引用的类型了 。

## 线程支持库

+ c++11开始支持
+ `static unsigned int hardware_concurrency()`返回支持的并发线程数。
+ 编译含有`std::thread`的程序需要加上`-pthead`

### 互斥

+ `std::mutex`
  + 线程占有 `mutex` 时，所有其他线程若试图要求 `mutex` 的所有权，则将阻塞（对于 `lock` 的调用）或收到 `false` 返回值（对于 `try_lock` ）.
  + 调用方线程在调用 `lock` 或 `try_lock` 前必须不占有 `mutex` 。
  + `std::mutex` 既不可复制亦不可移动。
  + 一般用 `std::lock_guard` 来管理 `mutex`

```c++
#include <mutex>

// gcc/libstdc++-v3/include/bits/std_mutex.h
// 都是调用 pthread 相应的函数
class mutex {
public:
    void lock(); 
    bool try_lock();
    void unlock();
    native_handle_type native_handle(); // 会以 pthread_mutex_t* 返回内部 mutex
};

class lock_guard {
public:
    explicit lock_guard(mutex_type& __m) :  _M_device(__m) {  _M_device.lock(); }
    ~lock_guard() { _M_device.unlock(); }
    lock_guard(const lock_guard&) = delete;
    lock_guard& operator=(const lock_guard&) = delete;
private:
    mutex_type&  _M_device; 
};
```

+ `void lock( Lockable1& lock1, Lockable2& lock2, LockableN&... lockn );`
  + 锁定给定的可锁 (Lockable) 对象 lock1 、 lock2 、 ... 、 lockn ，用免死锁算法避免死锁。

### future

```c++
#include <vector>
#include <thread>
#include <future>
#include <numeric>
#include <iostream>
#include <chrono>
 
void accumulate(std::vector<int>::iterator first,
                std::vector<int>::iterator last,
                std::promise<int> accumulate_promise)
{
    int sum = std::accumulate(first, last, 0);
    accumulate_promise.set_value(sum);  // 提醒 future
}
 
void do_work(std::promise<void> barrier)
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
    barrier.set_value();
}
 
int main()
{
    // 演示用 promise<int> 在线程间传递结果。
    std::vector<int> numbers = { 1, 2, 3, 4, 5, 6 };
    std::promise<int> accumulate_promise;
    std::future<int> accumulate_future = accumulate_promise.get_future();
    std::thread work_thread(accumulate, numbers.begin(), numbers.end(),
                            std::move(accumulate_promise));
    accumulate_future.wait();  // 等待结果
    std::cout << "result=" << accumulate_future.get() << '\n';
    work_thread.join();  // wait for thread completion
 
    // 演示用 promise<void> 在线程间对状态发信号
    std::promise<void> barrier;
    std::future<void> barrier_future = barrier.get_future();
    std::thread new_work_thread(do_work, std::move(barrier));
    barrier_future.wait();
    new_work_thread.join();
}
```



## 其它

+ C语言也有auto关键字，默认表示int类型，auto是一个隐含的默认存储类型，相对于static，register，extern等而言。

+ C语言不允许`函数重载`，因为编译时不会像C++一样对函数名做`name mangling`
  + c++中函数的名字有函数的namespace、函数名、参数共同组成。
  + `nm`命令可以查看符号表
  + `c++filt`命令可以得到原始的名字
  + C语言可以通过函数指针来实现重载，调用的时候选择指向不同的函数
  
+ 虚函数可以声明为`inline`，但具体能不能内联要看编译期是否能直接确定调用的具体的函数是哪一个，如`p->A::f()`，则会被内联。

+ C++的初始化列表的赋值顺序，是与C++类里面成员变量的声明顺序相关，与初始化列表里的顺序无关

  