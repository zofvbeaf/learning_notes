## 优化器

### BuildParseTree

+ [libpg_query](https://github.com/lfittl/libpg_query)：裁剪出的`postgresql`的语法解析和词法解析。

+ `BuildParseTree`：对`parser`生成的语法树进行解析，得到`SelectStatement`，其中记录了语句的各个组件。

  ```c++
  class SelectStatement : public SQLStatement {
    std::unique_ptr<TableRef> from_table;
    bool select_distinct;
    std::vector<std::unique_ptr<expression::AbstractExpression>> select_list;
    std::unique_ptr<expression::AbstractExpression> where_clause;
    std::unique_ptr<GroupByDescription> group_by;
  
    std::unique_ptr<SelectStatement> union_select;
    std::unique_ptr<OrderDescription> order;
    std::unique_ptr<LimitDescription> limit;
  };
  ```

### BindNameToNode

+ 主要是用于检查`SelectStatement`中涉及的列的合法性，并将列的属性（如属于哪一张表）填入各表达式。

### BuildPelotonPlanTree

#### Optimizer::InsertQueryTree

+ `QueryToOperatorTransformer::ConvertToOpExpression`：将`SelectStatement`转换为一棵树。树中节点如下

  + `LogicalQueryDerivedGet`：用于`from`中的子查询。
  + `LogicalInnerJoin`：默认对`from`中的表进行`Join`，解析到`from`时会生成`join`树，不带条件。
    + 如果`from`中显示的指定了哪一类`Join`，会生成对应类型的`Join`，如`LogicalLeftJoin`。
  + `LogicalGet`：递归解析`from`中的元素时会解析到底层的`TableRef`，生成此节点。
  + `LogicalFilter`：根据`where`中的谓词生成。having子句也会生成`LogicalFilter`。
  + `LogicalMarkJoin`：用于`IN`和`EXISTS`子链接。
  + `LogicalSingleJoin`：用于`expr op sublink`，`op`为比较运算符时生成。
  + `LogicalAggregateAndGroupBy`
  + `LogicalDistinct`：单独的节点，在`LogicalAggregateAndGroupBy`之后生成。
  + `LogicalLimit`：在`LogicalDistinct`后生成。

+ `OptimizerMetadata::RecordTransformedExpression`：

  + 将上一步生成的树中的节点逐个转换为`group_expression`，每个`group_expression`。
    + 封装层次：`Group <- GroupExpression <- Operator <- BaseOperatorNode <- <LogicalFilter...>`

#### Optimizer::GetQueryInfo

+ 提取`output_exprs`和`sort_exprs`作为后面优化的参考。

#### Optimizer::OptimizeLoop

+ 通过`TaskStack`来遍历树，具体规则应用顺序：

  + `BottomUpRewrite -> TopDownRewrite -> DeriveStats -> OptimizeGroup`

+ `BottomUpRewrite`和 `TopDownRewrite` 都是应用规则直接替换原来的节点。

+ `DeriveStats`

  + 通过栈实现`自底向上`的方式统计。主要统计以下信息：

    ```c++
    size_t num_rows;
    double cardinality;
    double frac_null;
    std::vector<double> most_common_vals,
    std::vector<double> most_common_freqs;
    std::vector<double> histogram_bounds;
    ```

+ `OptimizeGroup`

  + 先对`LogicalExpression`通过`OptimizeExpression`应用`TransformationRule`和`ImplementationRule`，得到多个`group`，并且`Expression`转换为了`PhysicalExpression`。
  + `OptimizeExpression`
    + 对当前`GroupExpression`添加`ApplyRule`，规则为所有`TransformationRule`和`ImplementationRule`。
    + 对当前的`child Group`调用`ExploreGroup`。
  + `ExploreGroup`
    + 对`Group`内的所有`LogicalExpressions`调用`ExploreExpression`。
  + `ExploreExpression`
    + 对当前`GroupExpression`添加`ApplyRule`，规则为所有`TransformationRule`和`ImplementationRule`。并且此`ApplyRule`设置了`explore_only`标志。
    + 对当前的`child Group`调用`ExploreGroup`。
  + `ApplyRule`
    + 每次对以当前`GroupExpression`为根节点的子树应用规则。
    + 通过`GroupExprBindingIterator`遍历子节点的`Group`，但是每次`Next()`返回都是当前`GroupExpression`，只是子节点在每次调用`HasNext()`时变化。
    + 通过`GroupBindingIterator`遍历其中的`LogicalExpression`。
    + 在应用`TransformationRule`后会重新调用`DeriveStats`。
    + 应用规则后得到的新的子树的根节点`GroupExpression`会被放到原来的`GroupExpression`所在的`Group`中，其新生成的子节点都会位于不同的`Group`。
  + `OptimizeInputs`
    + `ChildPropertyDeriver`：有的算子可能会要求前置节点的数据有序，如（`PhysicalSortGroupBy`），将此种属性下推到子节点。
    + 选择每个`Group`中开销最小的`GroupExpression`作为最佳节点，之后转换为`PhysicalPlan`节点。

## 优化规则

### TransformationRule

+ `InnerJoinCommutativity`

<figure center class="half">
    <img src="https://ws1.sinaimg.cn/large/77451733gy1g56jthhkyfj205104d745.jpg">
    ----->
    <img src="https://ws1.sinaimg.cn/large/77451733gy1g56jtpgu3fj205004a745.jpg">
</figure>

+ `InnerJoinAssociativity`

<figure center class="half">
    <img src="https://ws1.sinaimg.cn/large/77451733gy1g56ju4hv6yj206m06xglk.jpg">
    ----->
    <img src="https://ws1.sinaimg.cn/large/77451733gy1g56juc2sxdj206k072q2w.jpg">
</figure>

### ImplementationRule

+ `LogicalGroupByToHashGroupBy`
+ `LogicalAggregateToPhysical`
+ `GetToSeqScan`
+ `LogicalQueryDerivedGetToPhysical`
+ `InnerJoinToInnerNLJoin`
+ `InnerJoinToInnerHashJoin`
+ `ImplementDistinct`
+ `ImplementLimit`

### RewriteRule

+ `Bottom Up`：`UNNEST_SUBQUERY`

  + `PullFilterThroughMarkJoin`
    
    <figure center class="half">
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56jurtkd6j206606v749.jpg">
        ----->
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k1bb8hcj204w06zq2w.jpg">
    </figure>
    
  + `PullFilterThroughAggregation`
  
    <figure center class="half">
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k21ocz1j207a06xweg.jpg">
        ----->
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k2ddet6j207809ljrf.jpg">
    </figure>
    
  + `MarkJoinToInnerJoin`：即算子转换。
  
+ `Top Down`：`PREDICATE_PUSH_DOWN`

  + `PushFilterThroughJoin`

    <figure center class="half">
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k2neupnj205506xmx3.jpg">
        ----->
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k2u35vcj207c074q2x.jpg">
    </figure>
    
  + `PushFilterThroughAggregation`
  
   <figure center class="half">
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k34tz12j20780720sp.jpg">
        ----->
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k3bz7j4j207b0700sp.jpg">
    </figure>
  
  + `CombineConsecutiveFilter`
  
   <figure center class="half">
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k3oujdoj203f0713ye.jpg">
        ----->
        <img src="https://ws1.sinaimg.cn/large/77451733gy1g56k3vpm22j203j04amwz.jpg">
    </figure>
  
  + `EmbedFilterIntoGet`：即选择下推。

## 其它

+ `visitor`模式在遍历子树时较方便。

### 已做的优化

#### 等价谓词重写

+ 表达式预计算
+ `between and`转化
+ `IN(1, 2, 3)` 转 `OR`
+ `OR`和`ANY`也可以互转，但要看底层执行是否对`ANY`有优化，比如建了索引或哈希表。
+ `a > ANY (1, 2)` 转为 `a > 1`
+ `NOT (a != b)` 转为 `a = b`

#### 外连接消除

+ 目的：将外连接转换为内连接，从而更好地选择表的连接顺序。
+ 右外连接一般会转为左外连接处理。
+ 当外连接对应会补`NULL`的列在`where`中去掉了NULL的行时(如：`where col IS NOT NULL`)，可以转换为内连接。

#### 嵌套连接消除

+ 如果连接表达式只包括内连接，括号可以去掉

+ 如果连接表达式包括外连接，括号不可以去掉，至多能进行的就是外连接消除

  ```sql
  SELECT * 
  FROM t1 LEFT JOIN (t2 LEFT JOIN t3 ON t2.b = t3.b) ON t1.a = t2.a
  WHERE t1.a > 1
  ```

#### 连接消除

+ 主外键关系的表进行的连接，可消除主键表，这不会影响对外键表的查询。

+ 唯一键作为连接条件，三表内连接（中间表的列只作为连接条件）可以去掉中间表。

+ 其他一些特殊形式，可以消除连接操作

  ```sql
  SELECT MAX(a1) FROM A, B;
  SELECT DISTINCT a1 FROM A, B;
  SELECT a1 FROM A, B GROUP BY a1;
  ```

#### 子查询合并（Coalescing）

+ 含相同语句块的几个语句求OR或AND，可合并为一个语句。

#### 子查询展开（Unnesting）

+ 即子查询上拉或反嵌套

+ 如果子查询中出现了`AGG`，`GROUP BY`，`DISTINCT`子句，则子查询只能单独求解，不可以上拉。

+ 如果子查询只是简单的`SPJ`，则可以上拉。

+ 子查询上拉的前提是上拉后不能带来多余的元组。

+ 仅`EXISTS`的相关子查询可以上拉

+ `IN`类型

  ```sql
  outexpr IN (SELECT inexpr FROM ... WHERE subquery_where)
  -- 可被优化为
  EXISTS (SELECT 1 FROM ... WHERE subquery_where AND outexpr = inexpr)
  ```

  + 相当于被优化为`semi-join`。

  + **前提**：

    + `outexpr`和`inexpr`不能为`NULL`
    + 不需要从结果为FALSE的子查询中区分`NULL`

  + 若仅`outexpr`是非`NULL`的，则转换为

    ```sql
    EXISTS (SELECT 1 FROM ... WHERE subquery_where AND (outexpr = inexpr OR inexpr IS NULL))
    ```

+ `ALL/ANY/SOME`

  + `SOME`与`ANY`意义相同
  + `=ANY`与`IN`相同
+ `val > ALL(SELECT ...)` 等价于 `val > MAX(SELECT ...)`
  
+ `EXISTS`

  + 相关子查询会上拉