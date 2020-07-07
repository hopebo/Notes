## MySQL 优化器优化过程

1. 优化 where 条件
   1. 传递性建立多个等值条件，从这些等值条件消除可以推断出的条件
   2. 用常量替换某一列
   3. 消减一定为 true 或者 false 的条件

2. make_join_plan()

   ```c++
   /**
     Calculate best possible join order and initialize the join structure.
   
     @return true if success, false if error.
   
     The JOIN object is populated with statistics about the query,
     and a plan with table order and access method selection is made.
   
     The list of tables to be optimized is taken from select_lex->leaf_tables.
     JOIN::where_cond is also used in the optimization.
     As a side-effect, JOIN::keyuse_array is populated with key_use information.
   
     Here is an overview of the logic of this function:
   
     - Initialize JOIN data structures and setup basic dependencies between tables.
   
     - Update dependencies based on join information.
   
     - Make key descriptions (update_ref_and_keys()).
   
     - Pull out semi-join tables based on table dependencies.
   
     - Extract tables with zero or one rows as const tables.
   
     - Read contents of const tables, substitute columns from these tables with
       actual data. Also keep track of empty tables vs. one-row tables.
   
     - After const table extraction based on row count, more tables may
       have become functionally dependent. Extract these as const tables.
   
     - Add new sargable predicates based on retrieved const values.
   
     - Calculate number of rows to be retrieved from each table.
   
     - Calculate cost of potential semi-join materializations.
   
     - Calculate best possible join order based on available statistics.
     
     - Fill in remaining information for the generated join order.
   */
   
   bool JOIN::make_join_plan() {}
   ```

3. 如果是常量表或者implicit group，移除select_distinct
4. 优化掉distinct，转换为group by。消除 order by