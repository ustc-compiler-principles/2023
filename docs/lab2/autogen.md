# IR 自动化生成

学生将在本阶段提供的实验框架下，利用访问者模式遍历抽象语法树，调用 Light IR C++ 库，实现 IR 自动化生成。

## 实验框架介绍

### 抽象语法树

!!! notes "分析树（Parse Tree）与抽象语法树（Abstract Syntax Tree）"

    分析树在语法分析的过程中被构造；抽象语法树则是分析树的浓缩表示，使用运算符作为根节点和内部节点，并使用操作数作为子节点。进一步了解可以阅读 [分析树和抽象语法树的比较](https://stackoverflow.com/questions/5026517/whats-the-difference-between-parse-trees-and-abstract-syntax-trees-asts)。

实验框架实现了 lab1 生成的分析树到 C++ 上的抽象语法树的转换。可以使用[访问者模式](./visitor_pattern.md)来实现对抽象语法树的遍历，[ast.hpp](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler/-/blob/master/include/common/ast.hpp)文件中包含抽象语法树节点定义。

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

### `CminusfBuilder` 类

`CminusfBuilder` 类定义在[cminusf_builder.hpp](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/src/cminusfc/cminusf_builder.cpp)文件中，`CminusfBuilder` 类中定义了对抽象语法树不同语法节点的 `visit` 函数，实验已给出了一些语法树节点的访问规则，其余的需要学生补充。

在`CminusfBuilder` 构造函数函数中，下列代码片段是对[Cminusf 语义](../common/cminusf.md#cminusf-的语义)中的 4 个预定义函数进行声明并加入全局符号表中，在生成 IR 时可从符号表中查找。我们的测试样例会使用这些函数，从而实现 IO。

```cpp
scope.enter();
scope.push("input", input_fun);
scope.push("output", output_fun);
scope.push("outputFloat", output_float_fun);
scope.push("neg_idx_except", neg_idx_except_fun);
```

`CminusfBuilder` 类使用成员 `context` 存储翻译时状态，下列代码片段是 `context` 的定义，学生需要为该结构体添加更多域来存储翻译时的状态。

```cpp
struct {
    // function that is being built
    Function *func = nullptr;
} context;
```

## 实验内容
阅读[Cminusf 语义](../common/cminusf.md#cminusf-的语义)，并根据语义补全 `include/cminusfc/cminusf_builder.hpp` 与 `src/cminusfc/cminusf_builder.cpp` 文件，实现 IR 自动产生的算法，使得它能正确编译任何合法的 Cminusf 程序，生成符合 [Cminusf 语义](../common/cminusf.md#cminusf-的语义)的 IR。

**友情提示**：

1. 请比较通过 `cminusfc` 产生的 IR 和通过 clang 产生的 IR 来找出可能的问题或发现新的思路
2. 使用 GDB 进行调试来检查错误的原因
3. 我们为 `Function`、`Type` 等类都实现了 `print` 接口，可以使用我们提供的 [logging 工具](../common/logging.md) 进行打印调试。
4. 对于 C++ 不熟悉的学生可以参考[C++ 简介](../common/simple_cpp.md)

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

#### 情况一：生成 IR 文件

!!! note

    为了让 `cminusfc` 在 `$PATH` 中，一定要 `sudo make install`。

```shell
# 假设 cminusfc 的路径在你的 $PATH 中，并且你现在在 test.cminus 文件所在目录中
$ cminusfc test.cminus -emit-llvm
```

此时会在同目录下生成同名的 .ll 文件，在这里即为 `test.ll`。

#### 情况二：生成可执行文件

上面生成的 .ll 文件用于阅读，如果需要运行，需要调用 clang 编译链接生成二进制文件 `test`

```shell
$ clang -O0 -w -no-pie test.ll -o test -lcminus_io
```

!!! note

    上面的命令编译了 `test.ll`，并链接了 `cminus_io` 库。

**测试**
<!-- TODO: 把 general 加上去 -->

自动测试脚本和所有测试样例都是公开的，它在 `tests/2-ir-gen/autogen` 目录下，使用方法如下：

```sh
# 在 tests/2-ir-gen/autogen 目录下运行：
$ python3 ./eval_lab2.py
$ cat eval_result
```

测试结果会输出到 `tests/2-ir-gen/autogen/eval_result`。
