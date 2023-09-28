# IR 自动化生成

学生将在本阶段提供的自动化 IR 生成框架下，调用 LightIR C++ 库，实现 IR 自动化生成。

## 实验框架介绍

### 抽象语法树

为了便于大家进行实验，实验框架已经实现了 lab1 生成的分析树（parse tree）到 C++ 上的抽象语法树（Abstract Syntax Tree，AST）的转换。我们可以使用[访问者模式](./visitor_pattern.md)来实现对抽象语法树的遍历，[ast.hpp](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/include/common/ast.hpp)文件中包含抽象语法树节点定义，在[cminusf_builder.hpp](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/src/cminusfc/cminusf_builder.cpp) CminusfBuilder 类定义了不同语法树节点的 `visit` 函数，需要在此实验中调用 LightIR C++ 库补全 `visit` 函数生成 IR 的规则，来实现 IR 的自动化生成。

!!! notes "分析树与抽象语法树"

    分析树在语法分析的过程中被构造；抽象语法树则是分析树的浓缩表示，使用运算符作为根节点和内部节点，并使用操作数作为子节点。进一步了解可以阅读 [分析树和抽象语法树的比较](https://stackoverflow.com/questions/5026517/whats-the-difference-between-parse-trees-and-abstract-syntax-trees-asts。

<!-- ### Cminusf 预定义函数

Cminusf 语义中提到包含四个预定义的函数 `input`、`output`、`outputFloat` 和 `neg_idx_except`，四个预定义函数的实现在 `src/io` 目录下，在编译过程中被编译成 `cminus_io.a` 静态库，使用四个预定义函数的 Cminusf 程序，在被实验编译器编译成可执行文件时，需要链接 `cminus_io.a` 静态库。 -->

### 符号表 Scope

在 `include/cminusf_builder.hpp` 中，我们定义了一个用于存储作用域的类`Scope`。它的作用是辅助我们在遍历语法树时，管理不同作用域中的变量。它提供了以下接口：

```cpp
// 进入一个新的作用域
void enter();
// 退出一个作用域
void exit();
// 往当前作用域插入新的名字->值映射
bool push(std::string name, Value *val);
// 根据名字，寻找到值
Value* find(std::string name);
// 判断当前作用域是否是全局作用域
bool in_global();
```

你需要根据语义合理调用`enter`与`exit`，并且在变量声明和使用时正确调用`push`与`find`。

在类`CminusfBuilder`中，有一个`Scope`类型的成员变量`scope`，在类`CminusfBuilder`初始化函数中，将 `input`、`output` 等 Cminusf 预定义函数加入了全局作用域中，这相当于为预定义函数进行声明。

### `CminusfBuilder` 类


`CminusfBuilder`构造函数。。。预定义函数

翻译时状态 Context
在 `include/cminusf_builder.hpp` 中，我们还定义了一个用于存储翻译过程中状态的结构体`Context`，里面我们已经定义了一个 `func` 域，存储翻译过程中当前的函数。你还需要为这个结构体添加新的域来完成所有的翻译过程。

## 实验内容

补全 `gitlab/src/cminusfc/cminusf_builder.cpp` 文件中定义的 `CminusfBuilder` 类不同语法树节点的 `visit` 函数，实现 IR 自动产生的算法，使得它能正确编译任何合法的 Cminusf 程序，生成符合 [Cminusf 语义](../common/cminusf.md#cminusf-的语义)的 IR。

**友情提示**：

1. 简要阅读 `cminusf_builder.hpp` 和其他头文件中定义的函数和变量，理解项目框架也可以帮助完成实验。
2. 请比较通过 `cminusfc` 产生的 IR 和通过 clang 产生的 IR 来找出可能的问题或发现新的思路
3. 可以使用 GDB 等软件进行动态分析，可以单步调试来检查错误的原因

## 实验要求

### 仓库目录结构

与该阶段有关的文件如下。

```
.
├── CMakeLists.txt
├── include                             <- 实验所需的头文件
│   ├── ...
|   ├── cminusfc
|   |    └── cminusf_builder.hpp        <- 该阶段需要修改的文件
│   ├── lightir/*
│   └── common
│        ├── ast.hpp
│        ├── logging.hpp
│        └── syntax_tree.h
├── src
│   ├── ...
│   └── cminusfc
│       ├── cminusfc.cpp                <- cminusfc 的主程序文件
│       └── cminusf_builder.cpp         <- 该阶段需要修改的文件
└── tests
    ├── ...
    └── 2-ir-gen
        └── autogen
            ├── testcases                <- 助教提供的测试样例
            ├── answers                  <- 助教提供的测试样例
            └── eval_lab2.py             <- 助教提供的测试脚本
```

### 编译、运行和评测

**编译**

```shell
$ cd 2023ustc-jianmu-compiler
$ mkdir build
$ cd build
# 使用 cmake 生成 makefile 等文件
$ cmake ..
# 使用 make 进行编译
$ make
# 安装以链接 libcminus_io.a
$ sudo make install
```

如果构建成功，你会在 `build` 文件夹下找到 `cminusfc` 可执行文件，它能将 cminus 文件输出为 IR 文件，编译成二进制可执行文件。

**运行**

我们在 `tests/testcases_general` 文件夹中准备了一些通用案例。当需要对 `.cminus` 单个文件测试时，可以这样使用：

```sh
# 假设 cminusfc 的路径在你的$PATH中
# 1. 利用构建好的 Module 生成 test.ll
# 注意，如果调用了外部函数，如 input, output 等，则无法使用lli运行
cminusfc test.cminus -emit-llvm

# 假设libcminus_io.a的路径在$LD_LIBRARY_PATH中，clang的路径在$PATH中
# 1. 利用构建好的 Module 生成 test.ll
# 2. 调用 clang 来编译 IR 并链接上静态链接库 libcminus_io.a，生成二进制文件 test
cminusfc test.cminus
```

**测试**

自动测试脚本和所有测试样例都是公开的，它在 `tests/2-ir-gen` 目录下，使用方法如下：

```sh
# 在 tests/2-ir-gen 目录下运行：
python3 ./eval_lab2.py
```

测试结果会输出到同文件夹的 `eval_result` 下。
