# Efficiently Compiling Efficient Query Plans for Modern Hardware

## 摘要
- 传统的迭代器模型虽然灵活简单，但是在现代CPU上缺少局部性和频繁的指令错误预测，导致性能下降。尽管通过向量化等技术能够缓解这一状况，但是性能始终比不上手写的C++代码。
- 因此，我们使用LLVM编译器框架，将一个查询翻译转化为完整有效的机器码。

## 简介
- 大多数数据库将查询转化为一个物理的算术表达式，然后开始计算表达式算子的结果。每一个算子都从它的输入计算产生一个输出结果，允许上层算子通过next函数来调用获取本算子的输出，叫做火山迭代器的风格。
- 这样一种模型在IO能力是瓶颈的情况下，是可以很好的work的。但是随着磁盘性能的提升，IO不再是瓶颈，瓶颈转而变成了CPU时，这样的架构就不适用了。
- 虚函数和函数指针的调用比普通调用更耗时。

### 流水线定义
流水线意味着一个算子能够不经过拷贝或者物化操作，直接可以将数据传递给它的父算子。

### 文中方法与已经现存方法在以下几个方面的不同：
1. 查询处理过程不再是以算子为中心，而是以数据为中心，因此我们可以尽量将数据在CPU寄存器中保持尽可能持久的时间。算子的边界被打破。
2. 数据不再是由算子的pull模型，而是push模型。这样增强了代码和数据的局部性。

## 查询优化器
### pipeline-breaker
>we first give a definition of pipeline-breaker that is more restrictive than in standard database systems: An algebraic operator is a *pipeline breaker* for a given input side if it takes an incoming tuple out of the CPU registers. It is a *full pipeline breaker* if it materializes all incoming tuples from this side before continuing processing.

- 将数据分发到内存中也是一个打破流水线的操作，尽量将所有的数据都在CPU寄存器中保存的更久一点。

### 编译算术操作符
- 将一个查询分成多个细小的部分，因此每一个部分都可以将数据保存在CPU寄存器中，只有当需要获取新的tuple或者物化结果时才需要访问内存。
- 有着更好的代码局部性，因为小的代码块通过循环工作在大量的数据上。
- 从编译器的角度出发，每一个操作算子提供了像迭代器模型一样的接口`produce()/consume(attributes,source)`, 差别在于这些function都只是在code generation的阶段被使用。

## Code Generation
- 直接生成C++代码的编译时间太长。
- 因此选择LLVM(Low Level Vitrual Machine)编译器框架来生成可移植的汇编码，然后使用LLVM提供的JIT(Just-In-Time，当一段代码即将第一次被执行时被编译，为动态编译的一种)来直接执行。
- 整个查询中生成的机器码中，复杂数据结构和磁盘操作是C++写的，预编译的；齿轮是动态生成的。
- LLVM和C++可以容易的混合使用，因为它们可以直接调用彼此的方法。

### 分支预测
- 分支预测非常高效，只要分支大概率总是正确或者总是错误。如果一个判断是50%的概率，那么会很大影响分支预测的性能。
- 可以采取将判断条件分离的方式，使得每个判断条件都有较大概率为确定结果。
