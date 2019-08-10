## orca

### 传统的查询优化器

+ `postgresql`：
  + 整个查询计划用一个大的类表示，成员与`select`语句的各个子句一一对应；
  + 固定的优化流程。
  + `tpch`第二条语句的子链接，不会加`group by`再做`semi-join`，也就是不会上拉，所以每次执行都要很久。

### gporca整体优化流程

+ 输入: 要优化的 要求
+ 把 `expr` 插入到`memo` 中, 产生一些 `Group` 和一个 `Gexpr`. 通常来说就是把 原来`expr`中每个节点变成一个`Group`.
+ **Exploration:** 递归地生成逻辑上等价的`Gexpr` (详见后文). 此过程中会在已 有的`Group`中插入新的`Gexpr`, 也可能产生很多新的`Group`.
+ 为每个`Group`导出Scalar/Logical Property 和 `Statistics`.(每`Group` 内相同, 共享)
+ **Implementation:** 递归地为每个非`Scalar Group` 内的每个`gexpr (Logical)` 生成对应的`Physical gexpr`, 仍然放在相同的`Group`中.此时每个非`Scalar Group`内将同时有`logical gexpr` 和`physical gexpr`. (后面`logical gexpr` 基本不再用到)
+ **Optimization:** 详见下文。主要包含两部分：添加了`Enforcer`，计算代价并记录最优`gexpr`。
+ 优化完成，提取最优`expr`。