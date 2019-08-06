## postgresql 查询引擎

+ 入口：`src/backend/tcop/postgres.c:exec_simple_query()`

### parser

+ `src/backend/tcop/postgres.c:pg_parse_query() -> raw_parser()`
+ 返回 `List*`，第一个节点即为`RawStmt *`
+ 具体结构见语法树文件

### pg_analyze_and_rewrite

+ 规则写在`pg_rules`和`pg_reswrite`系统表中

#### parse_analyze

+ 调用`transformTopLevelStmt`将`raw_parser()`的结果中的`RawStmt`转换为`Query tree`

  ```c
  typedef struct Query
  {
      //上面还有节点类型，是否存在相关子句
      List       *cteList;        /* WITH 子句 */
      List       *rtable;         /* list of range table entries */
      FromExpr   *jointree;       /* table join tree (FROM and WHERE clauses) */
      List       *targetList;     /* target list (of TargetEntry) */
      List       *returningList;  /* return-values list (of TargetEntry) */
      List       *groupClause;    /* a list of SortGroupClause's */
      Node       *havingQual;     /* qualifications applied to groups */
      List       *windowClause;   /* 窗口函数子句链表 */
      List       *distinctClause; /* a list of SortGroupClause's */
      List       *sortClause;     /* a list of SortGroupClause's */
      
      Node       *limitOffset;    /* limit的offset子句 */
      Node       *limitCount;     /* limit的个数*/
      Node       *setOperations;  /* 是否为多个SQL UNION/INTERSECT/EXCEPT query */
  } Query;
  ```
  + `from`成员的表示：可以表示单独的表或子查询，或连接操作的结果，表示被查询的对象，查询优化前的表示

    ```c
    typedef struct RangeTblEntry
    {
        //普通表
        Oid         relid;          /* OID of the relation */
        char        relkind;        /* relation kind (see pg_class.relkind) */
        struct TableSampleClause *tablesample;     /* sampling info, or NULL */
        
        //子查询
        Query      *subquery;       /* the sub-query */
        bool        security_barrier;       
        /* is from security_barrier view?如果是视图展开的子查询，PostgreSQL不做优化 */
        
        //连接类型
        JoinType    jointype;       /* type of join */
        List       *joinaliasvars;  /* list of alias-var expansions */
    } RangeTblEntry;
    ```

#### pg_rewrite_query

+ 对`select`语句，主要是递归找语句中出现的表进行视图重写。

### pg_plan_queries

+ 调用`pg_plan_query()`对`rewrite`后的`Query`生成`PlannedStmt`。

+ 调用`backend/optimizer/plan/planner.c:planner()`，即`optimizer`，如果想自定义优化器，可以用 `planner_hook` 指向它。默认调用`standard_planner()`

+ 调用`subquery_planner()`递归优化`SELECT`语句，因为可能存在子查询或子链接。
  + 创建`PlannerInfo`存放优化过程中的信息，如：查询层级，基表及`Join`关系信息，代价估算信息等。
  
  + `SS_process_ctes`：递归调用`subquery_planner()`处理`CTE`表达式。
  
  + `pull_up_sublinks`：上拉子链接
    
    + 调用`pull_up_sublinks_jointree_recurse`
      + 递归遍历`jointree`，更新对应的`relid`位图
        + 对`RangeTblRef`直接返回
        + 对`FromExpr`分别遍历处理`fromlist`和`quals`
        + 对`JoinExpr`分别遍历处理左右操作数和`quals`
        + 对于`quals`调用`pull_up_sublinks_qual_recurse`
          + 内部会执行`ANY`、`EXISTS`等转变为`JOIN`的操作（当判断条件符合时）
          + `ANY`上拉后，创建一个名为`ANY_SUBQUERY`的`RangeTblEntry`加入父查询的`rtable`中
    + `foo op ANY (sub-select)`转变为`semi-join`，必须非相关
    + `EXISTS`转变为`semi-join`，将子链接中的关联谓词上拉，做`semi-join`
    + `NOT EXISTS`转变为`anti-semi-join`，将子链接中的关联谓词上拉，做`anti-semi-join`
    
  + `pull_up_subqueries`
  
    + 主要是调用`pull_up_simple_subquery`将子查询中的基表上拉到父查询
  
  + `preprocess_expression`，对`Query`的各个部分调用此函数
  
    + `eval_const_expressions`
      + 表达式预处理，包括表达式预计算，`bool`值相关表达式化简。
    + `canonicalize_qual`
      + 使用`OR`分配律化简逻辑表达式函数，如：`((A AND B) OR (A AND C))`转换为`(A AND (B OR C))`
    + `SS_process_sublinks`
      + 用于求解表达式中的子链接，调用`make_subplan`得到`SubPlan`
  
  + `reduce_outer_joins`
  
    + 消除外连接
  
  + `grouping_planner`
  
    + 如果有集合操作，先处理集合操作
  
    + `preprocess_groupclause`
  
      + **调整`group by`中列的顺序尽量和`order by`相同**，两层循环实现
  
    + `preprocess_targetlist`
  
      + 对`INSERT`或`UPDATE`语句会补充列信息。
      + 对`SELECT`，无特殊处理。
  
    + `preprocess_minmax_aggregates`对于没有`group by`的`minmax`操作构建最优（索引）查询执行计划。
  
    + `query_planner`：生成查询路径
  
      + `setup_simple_rel_arrays`：收集基表信息
  
        + 创建`RangeTblEntry*`数组`simple_rte_array`，存入查询语句涉及的`RangeTblEntry`
        + 创建`RelOptInfo*`数组`simple_rel_array`，后续优化过程中会逐步填入。
  
      + `add_base_rels_to_query`
  
        + 递归遍历`jointree`，构建`RelOptInfo`数组，
        + 遍历到`RangeTblRef`时调用`build_simple_rel`
          + 创建`RelOptInfo`，初始化该基表的元数据信息，如：`min_attr`，`max_attr`，`tuples`等
  
      + `build_base_rel_tlists`：设置目标列
  
        + `add_vars_to_targetlist`：将`targetlist`中的列添加到所属的`RelOptInfo`的`reltargetlist`中。
  
      + `deconstruct_jointree`
  
        + 生成`joinlist`，即串成链表。
        + 递归遍历`jointree`，将约束条件填入相关的`RelOptInfo`，和两表有关的条件放在连接得到的临时节点上。
        + 收集所有`relid`到`qualscope`中
        + `distribute_qual_to_rels`
          + 把约束下推到`RelOptInfo`（可能为单表，可能为多表`join`）上，并且会生成隐含的等价条件进行下推。
  
      + `make_one_rel`：物理优化开始，寻找最优的查询访问路径 
  
        + 代价估算模型为：**总代价 = 启动代价 + IO代价 + CPU代价**
  
        + `set_base_rel_sizes`：
  
          + 查系统表，如`pg_stats`和`pg_class`，设置基表的物理参数，如元组数和数据分布情况、索引等
          + 基表可能也有部分索引（对部分数据建立的特殊索引）
          + `set_baserel_size_estimates`
            + 大致是每条元组的代价乘以元组数，另外也会考虑每页读取的开销，索引扫描的开销等
            + 另外也会根据谓词条件计算选择率
            + 基表查询有三种方式：
              + 顺序访问
              + 索引访问
              + 直接TID访问，访问指定行
  
        + `set_base_rel_pathlists`
  
          + 为每个基表关系生成一个`RelOptInfo`结构，一并生成的访问路径放在其`pathlist`成员中，此处生成的访问路径是单个表的最优单表扫描方式。
          + `set_cheapest`
            + 基表扫描会选择三种扫描方式中代价最小的
            + 若是表连接后的中间关系，找的是形成这个中间关系的多种连接方式中花费代价最小的
  
        + `make_rel_from_joinlist`
  
          + 所有的基本关系的路径由`set_base_rel_pathlists`生成后，就可以通过递归调用`make_rel_from_joinlist`函数把基本关系分别连接起来（选择不同的连接方式生成中间节点，选择不同的连接顺序将基本关系的叶子节点组成一棵二叉树，对上层返回代价最小的路径），形成最终路径。
  
          + 示例语句：
  
            ```sql
            SELECT * FROM A, B, C, D WHEREA.a = B.a AND A.a = C.a AND A.a = D.a;
            ```
  
          + 根据`joinlist`的长度判断`levels_needed`，示例语句中则为`4`。
  
          + 当`levels_needed > 1`时会选择不同的查询优化算法
  
            ```c
            if (join_search_hook) // 有用户自定义的算法
            	return (*join_search_hook) (root, levels_needed, initial_rels);
            else if (enable_geqo && levels_needed >= geqo_threshold)  // >= 12
            	return geqo(root, levels_needed, initial_rels); // 遗传算法
            else // 动态规划算法
            	return standard_join_search(root, levels_needed, initial_rels);
            ```
  
          + `standard_join_search`
  
            ```c
            root->join_rel_level[1] = initial_rels; // 初始化第一层为 {A},{B},{C},{D}
            
            for (lev = 2; lev <= levels_needed; lev++) {
                join_search_one_level(root, lev); // 递归搜索第更高层的结果
                foreach(lc, root->join_rel_level[lev]) {
                    rel = (RelOptInfo *) lfirst(lc);
                    set_cheapest(rel); // 选择当前层的最优路径
                }
            }
            // 递归中间结果示例:
            // 第一层为 {A},{B},{C},{D}
            // 第二层为 {AB},{AC},{AD} 等
            // 第三层为 {ABC},{ACD},{ABD} 等
            // 第四层为 {ABCD} 的某种顺序，如还可能为 {CADB}
            ```
  
            + `join_search_one_level`
  
              ```c
              // 左深树的做法
              // 循环取出前一层的结果与第一层的结果构建join
              foreach(r, joinrels[level - 1]) {
              	RelOptInfo *old_rel = (RelOptInfo *) lfirst(r);
              	other_rels = list_head(joinrels[1]);
              	make_rels_by_clause_joins(old_rel,other_rels); 
                  // 内部调用 make_join_rel
              }
              // bushy plans 紧密树
              // 用第 k 层和 level-k 层做连接，构成第 level 层的关系
              for (k = 2;; k++) {
                  foreach(r, joinrels[k]) {
                      RelOptInfo *old_rel = (RelOptInfo *) lfirst(r);
                      other_rels = list_head(joinrels[other_level]);
                      // 判断两个表集合不是完全包含关系就做 make_join_rel
                  }
              }
              ```
  
              + `make_join_rel`
                + 两个表之间的`INNER JOIN`会生成两种方式，`A join B` 和 `B join A`
                + 内部调用`build_join_rel`生成`join`对应的`RelOptInfo`
                  + 生成之前会先判断路径已经生成过了，如`{A, B, C}`可能被`{A},{B, C}`生成，也可能被`{AB},{C}`生成
                + `populate_joinrel_with_paths`根据不同类型的`JOIN`会生成相应的路径，也会转换生成如`mergejoin`，`hashjoin`等。之后调用`add_paths_to_joinrel`生成路径，`join`路径的代价也在生成时进行计算
                  + `join`的开销
                    + `selectivity_left * selectivity_right * tuples_left * tuples_right`
                    + 对不同`join`计算类似，只是半连接的开销会认为较小，因为只要一个匹配即返回。
  
+ `create_plan`：在`subquery_planner`结束之后调用，位于standard_planner内，根据相应的节点类型递归创建相应`Plan`结构即可