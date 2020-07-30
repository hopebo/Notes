# An Overview of Query Optimization in Relational Systems

## 1 Introduction
数据库系统的两个核心组件是查询优化器和查询执行引擎。

查询执行引擎实现了一系列的物理算子（包括排序，顺序扫描，索引扫描，嵌套循环Join，sort-merge Join等），物理算子并不需要和关系算子一一对应，可以将物理算子看作是代码片段，共同组成执行查询的代码块。一个查询的执行可以抽象的看作是一个物理算子树，执行引擎的能力决定了可用的物理算子树的结构。

查询优化器的作用是给执行引擎提供输入，它将解析过的SQL查询作为输入，负责从可能的执行计划空间来产生一个高效的执行计划。可能的物理算子树的数量可能十分巨大，出于以下两个原因：
- 一个查询的算术表达式可以等价转换为很多相同的算术表示。
- 对于每一个算术表达式，可能实现的物理算子树也会有很多。

对于不同的物理算子树，执行的用时可能差别很大。查询优化器的责任就是找到相对高效的执行计划，可以看作是一个搜索问题。为了解决这一个问题，我们需要提供：
- 执行计划的一个搜索空间。
- 代价估算的技术，能够给搜索空间的每一个计划都计算出执行代价。
- 用于在搜素空间中寻找最佳计划的枚举算法。

一个好的查询优化器应该具备三个条件：（1）搜索空间中包含代价最小的计划。（2）代价估算技术是准确的。（3）枚举算法足够高效。

## 2 System-R Optimizer
代价估算模型给搜索空间中部分或全部的计划以bottom-up的方式计算出以下的信息：
1. 算子结点输出的数据流的大小。
2. 算子树结点需要在输出数据流上创建或者维护的顺序。
3. 操作算子以下部分的计划的执行代价。

这些依赖于：
1. 维护在关系或者索引上的一系列统计数据，包括关系上的数据页数量、索引页的数量和列上的不同值数量。
2. 估算选择率的公式和计算数据流大小的公式。
3. 估算CPU和IO代价的公式。

System-R的枚举算法采用动态规划与interesting order结合的方式进行，它认为最优的代价由最优的子表达式与新增的表达式组成。只有代表着相同的表达式，同时具有相同interesting order的操作符算子树才被直接比较。

## 3 Search Space
对查询语句的算术转换并不一定减少代价，所以转换规则都必须在基于代价估算的规则下进行。

在System-R系统中，只考虑线性排列的join方式。Inner join满足交换律和结合律。单边Outer join是非对称算子，不满足交换律，但是可以将inner join换在一块，outer join换在一块，这样inner join块内部是可以自由交换结合的。

当Group by与join结合在一块时，可以考虑将Group by下推先做，这样能大幅减少tuple的数量，再做join就会使代价大大降低。这样做的前提条件是，Group by列中包含列是另一个表Join列的外键。

### 3.1 Reducing multi-block queries to single-block
使用合并视图、合并嵌套子查询等方法可以将多个查询块合并为一个，这样就可以针对当个查询块采用更多优化手段，包括交换Join顺序以及结合等。

### 3.2 Using semijoin like techniques for optimizing multi-block queries
使用semojoin的技术来优化查询，既可以减少计算量，又可以匹配相关结果。

## 4 Statistics and cost estimation
代价估算就是决定那种算子树所消耗的资源最少，这些资源包括CPU时间、IO代价、内存、通信带宽或者这些的组合。基本的代价估算框架包括：
1. 收集已经存储的数据的统计数据。
2. 对于指定操作符，给出它的所有输入数据流的统计数据，能够计算出：
  1. 输出数据流的统计数据。
  2. 执行操作符的估算代价。

### 4.1 Statistical information on base data
1. Tuple的数量能够帮助我们确定数据扫描，join的代价以及他们需要的内存。
2. 表所使用到的物理页的数量。
3. 在列上的统计数据能够计算在列上的条件的选择率大小。这些统计信息在有索引的列上创建。

直方图也可以用来做统计数据的收集，包括等高直方图（落在每个直方范围内的数据数量是相同的），单值直方图（每个bucket仅代表一个值）。

### 4.2 Estimating statistics on base data
统计数据的计算通常是通过采样的方法来收集的。

### 4.3 Propagation of statistical information
统计数据需要在算子之间进行传播。例如，如果多个判断条件同时存在，会基于独立性假设来估算选择率。

*What is selectivity:*
> That’s where selectivity comes in. We’ve already been discussing selectivity, without naming it. Selectivity is a measure of how much the index narrows the search for values. There are two ways to express it:
>
>       1. Average selectivity related to the cardinality. So if there are, say, 10,000 distinct first names in the United States (a number I’m having a hard time getting facts to support), the average selectivity of an index on the first name is 10,000/249,000,000=0.00004.
>
>       2. The selectivity of a specific value is the number of rows with that value, divided by the total number of rows. The selectivity of “Baron Schwartz” is much better than “John Smith.”
>
>A lower selectivity value is better: it means fewer rows to scan and filter. Oddly, though, we say an index is “highly” selective if it has a low selectivity value.

## 5 Enumeration architectures
一个枚举算法应该很好的适应由于新增转换规则或者新加物理算子而带来的搜索空间的变化。

Starburst和Volcano/Cascades算法都有着下列共同点：
1. 使用通用的代价函数和算子的物理属性。
2. 使用规则引擎能够允许对算子树进行转换。
3. 暴露出一些参数能够去调节系统行为。

### 5.1 Starburst
在IBM，使用结构化的表示方法去表示SQL查询，称为QGM(Query Graph Model), 一个box就表示一个查询块，box之间的弧线表示表的引用关系。规则转换由一对方法组成，第一个检查可应用的条件，第二个进行真实的转换。这种枚举算法是一种类似bottom-up的算法。

### 5.2 Volcano/Cascades
在这样的系统中，规则在全局范围内被使用来表示搜索空间。有两种规则被使用：
1. 转换规则将算术表达式转换为另一个。
2. 实现规则将算术表达式映射为操作算子树。

Volcano/Cascades按照Top-down的方式使用动态规划的算法来寻找最优。在优化过程中，可能应用逻辑转换规则、实现规则或者强制使数据流具有某种属性。

与Starburst系统不同的地方在于：
1. 并不使用两个独立的优化阶段，因为所有的转换都是基于算术和代价估算的。
2. 从算术到物理算子的映射出现在单个步骤中。
3. 采取的是一种目标驱动的方式，而不是链状向前的方式。
