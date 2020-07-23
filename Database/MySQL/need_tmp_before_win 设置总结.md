# need_tmp_before_win 设置总结

叙述按照代码流程顺序：

## join buffer 设置

1. 如果存在 join buffer 设置，那么对前一个表的 join buffer 随后会改变输出行的顺序，因此无法依赖第一次非常数表因为 index scan 而呈现有序的输出。会设置 simple_order = simple_group = false 。

   ```c++
   {
       /*
         If the hint FORCE INDEX FOR ORDER BY/GROUP BY is used for the first
         table (it does not make sense for other tables) then we cannot do join
         buffering.
       */
       if (!plan_is_const()) {
         const TABLE *const first = best_ref[const_tables]->table();
         if ((first->force_index_order && order) ||
             (first->force_index_group && group_list))
           no_jbuf_after = 0;
       }
   
       bool simple_sort = true;
       // Check whether join cache could be used
       for (uint i = const_tables; i < tables; i++) {
         JOIN_TAB *const tab = best_ref[i];
         if (!tab->position()) continue;
         if (setup_join_buffering(tab, this, no_jbuf_after)) DBUG_RETURN(true);
         if (tab->use_join_cache() != JOIN_CACHE::ALG_NONE) simple_sort = false;
         DBUG_ASSERT(tab->type() != JT_FT ||
                     tab->use_join_cache() == JOIN_CACHE::ALG_NONE);
       }
       if (!simple_sort) {
         /*
           A join buffer is used for this table. We here inform the optimizer
           that it should not rely on rows of the first non-const table being in
           order thanks to an index scan; indeed join buffering of the present
           table subsequently changes the order of rows.
         */
         simple_order = simple_group = false;
       }
     }
   }
   ```

2. 某个排序的 item 是 stored procedure 或者 user-defined function，导致消耗大的计算，将 simple_order 和 simple_group 设置为 false 。

   ```c++
   if (!plan_is_const() && order) {
       /*
         Force using of tmp table if sorting by a SP or UDF function due to
         their expensive and probably non-deterministic nature.
       */
       for (ORDER *tmp_order = order; tmp_order; tmp_order = tmp_order->next) {
         Item *item = *tmp_order->item;
         if (item->is_expensive()) {
           /* Force tmp table without sort */
           simple_order = simple_group = false;
           break;
         }
       }
     }
   ```

##8个条件下设置 need_tmp_before_win 为 true

1. ROLLUP 添加的行输出仅对 DISTINCT, WINDOW 和 ORDER BY 可见。如果 ROLLUP 和这些语法同时使用，需要使用临时表对 ROLLUP 的结果进行暂存。

2. 如果所有的表都是常量的，查询的结果保证仅有0或1行，那么接下来讨论的所有的语法（DISTINCT, ORDER BY, GROUP BY, WINDOWING, SQL_BUFFER_RESULT）都是没有意义的，所以不需要使用临时表。

3. 如果存在 GROUP BY 没有被决定为第一个表使用索引或排序，那么需要用临时表来计算聚合行。这是一个在 WINDOWING 之前的临时表。

4. 如果存在 DISTINCT 没有被决定为第一个表使用索引或者排序，那么就需要一个临时表作为输入，这是在没有 WINDOW 函数的前提下，由于这个语法在 WINDOW 函数之后发生，所以有 WINDOW 的情况下，可以使用最后一个 WINDOW 的临时表。

5. ORDER BY 语法同上。

6. 如果存在不同的 GROUP BY 和 ORDER BY 顺序，ORDER BY 需要临时表作为输入。

7. 如果用户想要去缓存结果，那么我们需要一个临时表，但是 WINDOW 函数和派生表的物化已经为我们创造了所需的临时表。

8. 如果第一个 WINDOW 步骤需要做排序，那么它会使用 filesort，能对一个单表进行排序，但是在对多表进行 join 的情况下，我们需要一个临时表。如果 GROUP BY 已经被优化掉了，那么 pre-windowing 的结果要么是0，要么是1，在这种情况下不需要排序。

   ```c++
     /*
       Check if we need to create a temporary table prior to any windowing.
   
       (1) If there is ROLLUP, which happens before DISTINCT, windowing and ORDER
       BY, any of those clauses needs the result of ROLLUP in a tmp table.
   
       Rows which ROLLUP adds to the result are visible only to DISTINCT,
       windowing and ORDER BY which we handled above. So for the rest of
       conditions ((2), etc), we can do as if there were no ROLLUP.
   
       (2) If all tables are constant, the query's result is guaranteed to have 0
       or 1 row only, so all SQL clauses discussed below (DISTINCT, ORDER BY,
       GROUP BY, windowing, SQL_BUFFER_RESULT) are useless and need no tmp
       table.
   
       (3) If there is GROUP BY which isn't resolved by using an index or sorting
       the first table, we need a tmp table to compute the grouped rows.
       GROUP BY happens before windowing; so it is a pre-windowing tmp
       table.
   
       (4) (5) If there is DISTINCT, or ORDER BY which isn't resolved by using an
       index or sorting the first table, those clauses need an input tmp table.
       If we have windowing, as those clauses are used after windowing, they can
       use the last window's tmp table.
   
       (6) If there are different ORDER BY and GROUP BY orders, ORDER BY needs an
       input tmp table, so it's like (5).
   
       (7) If the user wants us to buffer the result, we need a tmp table. But
       windowing creates one anyway, and so does the materialization of a derived
       table.
   
       See also the computation of Temp_table_param::m_window_short_circuit,
       where we make sure to create a tmp table if the clauses above want one.
       
       (8) If the first windowing step needs sorting, filesort() will be used; it
       can sort one table but not a join of tables, so we need a tmp table
       then. If GROUP BY was optimized away, the pre-windowing result is 0 or 1
       row so doesn't need sorting.
     */
   
     if (rollup.state != ROLLUP::STATE_NONE &&  // (1)
         (select_distinct || has_windows || order))
       need_tmp_before_win = true;
   
     if (!plan_is_const())  // (2)
     {
       if ((group_list && !simple_group) ||               // (3)
           (!has_windows && (select_distinct ||           // (4)
                             (order && !simple_order) ||  // (5)
                             (group_list && order))) ||   // (6)
           ((select_lex->active_options() & OPTION_BUFFER_RESULT) &&
            !has_windows &&
            !(unit->derived_table &&
              unit->derived_table->uses_materialization())) ||     // (7)
           (has_windows && (primary_tables - const_tables) > 1 &&  // (8)
            m_windows[0]->needs_sorting() && !group_optimized_away))
         need_tmp_before_win = true;
     }
   ```

## Temp table 用来做 Group by

```C++
  if (group_list &&
      (!(select_lex->active_options() & SELECT_BIG_RESULT || with_json_agg) ||
       (tab->quick() &&
        tab->quick()->get_type() == QUICK_SELECT_I::QS_TYPE_GROUP_MIN_MAX)) &&
      m_ordered_index_usage != JOIN::ORDERED_INDEX_GROUP_BY &&
      (tmp_table_param.quick_group ||
       (tab->table_list()->embedding &&
        tab->position()->sj_strategy == SJ_OPT_LOOSE_SCAN))) {
    need_tmp_before_win = true;
    simple_order = simple_group = false;
  }
```

