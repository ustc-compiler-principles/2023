# 访问者模式

### 简介

Visitor Pattern（访问者模式）是一种在 LLVM 项目源码中被广泛使用的设计模式。在本实验中，指的是**语法树**类有一个方法接受**访问者**，将自身引用传入**访问者**，而**访问者**类中集成了根据语法树节点内容生成 IR 的规则，下面我们将通过例子来帮助大家理解 Visitor Pattern 的调用流程。

!!! info

    有关 Visitor Pattern 的含义、模式和特点，可参考 [维基百科](https://en.wikipedia.org/wiki/Visitor_pattern#C++_example)。

### 实验内容
<!-- TODO: 重写 -->
在 `tests/2-ir-gen/warmup/calculator` 目录下提供了一个接受算术表达式，利用访问者模式，产生计算算数表达式的中间代码的程序，其中 `calc_ast.hpp` 定义了语法树的不同节点类型，`calc_builder.cpp` 实现了访问不同语法树节点 `visit` 函数。**阅读这两个文件和目录下的其它相关代码**，理解语法树是如何通过访问者模式被遍历的，并回答相应[思考题](./visitor_pattern.md#思考题)。

### 编译、运行

**编译**

```shell
$ cd 2023ustc-jianmu-compiler
$ mkdir build
$ cd build
# 使用 cmake 生成 makefile 等文件
$ cmake ..
# 使用 make 进行编译
$ make
```

如果构建成功，你会在 `build` 文件夹下找到`calc`可执行文件

**运行与测试**

```shell
# 在 build 目录下操作
$ ./calc
Input an arithmatic expression (press Ctrl+D in a new line after you finish the expression):
4 * (8 + 4 - 1) / 2
result and result.ll have been generated.
$ ./result
22
```

其中，`result.ll` 是 calc 产生的中间代码，`result` 是中间代码编译产生的二进制可执行文件，运行它就可以输出算数表达式的结果。

## 思考题

1. 分析 `calc` 程序在输入为 `4 * (8 + 4 - 1) / 2` 时的行为：

   1. 请画出该表达式对应的抽象语法树（使用 `calc_ast.hpp` 定义的语法树节点来表示，并给出节点成员存储的值），并给节点使用数字编号。
   2. 请给出示例代码在用访问者模式遍历该语法树时，访问者到达语法树节点的顺序。序列请按如下格式指明（序号为问题 1.a 中的编号）：3->2->5->1->1
