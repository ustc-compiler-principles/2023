# Light IR 预热

本阶段实验需要完成以下两部分

1. 阅读 [Light IR 简介](../common/LightIR.md) 了解 LLVM IR 与 Light IR 关系，认识 Light IR 的结构及指令语法。在 clang 翻译出的 IR 作为参考下，手动编写 IR 文件。

2. 了解并掌握 Light IR C++ 库的使用方法，通过调用 Light IR C++ 库提供的接口生成 IR 文件。

## 手工编写 IR 文件

### LLVM IR 介绍

根据[维基百科](https://zh.wikipedia.org/zh-cn/LLVM)的介绍，LLVM 是一个自由软件项目，它是一种编译器基础设施，以 C++ 写成，包含一系列模块化的编译器组件和工具链，用来开发编译器前端和后端。IR 的全称是 Intermediate Representation，即中间表示。LLVM IR 是 LLVM 项目定义的编译器中间代码。

LLVM IR 指令参考手册：[Reference Manual](https://llvm.org/docs/LangRef.html)

### Light IR 介绍

LLVM IR 的目标是成为一种通用 IR（支持包括动态与静态语言），因此 IR 指令种类较为复杂繁多。本课程从 LLVM IR 中裁剪出了适用于教学的精简的 IR 子集，并将其命名为 Light IR。

Light IR 指令参考手册：[Light IR 手册](../common/LightIR.md#ir-%E6%A0%BC%E5%BC%8F)

### clang 生成 LLVM IR

<!-- TODO: 重写 bash cat -->

LLVM IR 文件以 `.ll` 为文件后缀，clang 是 LLVM 工具链中的前端可实现从 C 语言向 LLVM IR 的翻译，操作流程如下

```shell
# 可用 clang 生成 C 代码对应的 .ll 文件
$ clang -S -emit-llvm gcd_array.c
# lli 可用来执行 .ll 文件
$ lli gcd_array.ll
# `$?` 的内容是上一条命令所返回的结果，而 `echo $?` 可以将其输出到终端中
$ echo $?
```

[gcd_array.c](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler/-/blob/master/tests/2-ir-gen/warmup/ta_gcd/gcd_array.c)是实验提供的例子，学生可以通过使用 clang 翻译示例，并查阅 [Light IR 手册](../common/LightIR.md#lightir-指令)来理解每条 LLVM IR 指令含义。

### 实验内容

<!-- TODO: 把 2023ustc-jianmu-compiler-ta 换成公开仓库 -->

实验在 `tests/2-ir-gen/warmup/c_cases/` 目录下提供了四个 C 程序： `assign.c`、 `fun.c`、 `if.c` 和 `while.c`。学生需要在 `test/2-ir-gen/warmup/stu_ll` 目录中，手动使用 LLVM IR 将这四个 C 程序翻译成 IR 代码，得到 `assign_hand.ll`、`func_hand.ll`、`if_handf.ll` 和 `while_hand.ll`，可参考 `clang -S -emit-llvm` 的输出。

### 运行、测试

**运行**：.ll 文件的运行与测试参考[运行 IR 文件](./warmup.md#clang 生成 LLVM IR)

**测试**：lli 运行 `tests/2-ir-gen/warmup/stu_ll` 目录下四个.ll 文件，并通过检查返回值来判断 .ll 文件的正确性。

## 使用 Light IR C++ 库生成 IR 文件

### Light IR C++ 库介绍

LLVM 项目提供了辅助 IR 生成的 C++ 库，但其类继承关系过于复杂，并且存在很多为了编译性能的额外设计，不利于学生理解 IR 抽象。因此实验依据 LLVM 的设计，为 Light IR 提供了配套简化的 C++ 库。与 LLVM IR C++ 库相比，Light IR C++ 库仅保留必要的核心类，简化了核心类的继承关系与成员设计，且给学生提供与 LLVM 相同的生成 IR 的接口。

Light IR IR C++ 库参考手册：[Light IR cpp APIs](../common/LightIR.md#c-apis)

### 使用 Light IR C++ 库生成 IR 示例

阅读样例 [gcd_array.c](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler/-/blob/master/tests/2-ir-gen/warmup/ta_gcd/gcd_array.c), [gcd_array_generator.cpp](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler/-/blob/master/tests/2-ir-gen/warmup/ta_gcd/gcd_array_generator.cpp)。结合该样例的注释与 [Light IR C++ 库](../common/LightIR.md#)章节，掌握使用 Light IR C++ 库生成 IR 的方法

### 实验内容

实验在 `tests/2-ir-gen/warmup/c_cases/` 目录下提供了四个 C 程序。学生需要在 `tests/2-ir-gen/warmup/stu_cpp/` 目录中，参考上面提供的 `gcd_array_generator.cpp` 样例，使用 Light IR C++ 库，编写 `assign_generator.cpp`、`fun_generator.cpp`、`if_generator.cpp` 和 `while_generator.cpp` 四个 cpp 程序。这四个程序运行后应该能够生成 `tests/2-ir-gen/warmup/c_cases/` 目录下四个 C 程序对应的 .ll 文件。

### 编译、运行、测试

!!! note

    如果你在 `cmake ..` 一步遇到如下报错`Target "IR_lib" links to target "ZLIB::ZLIB" but the target was not found.`，请使用 `sudo apt install zlib1g-dev` 安装 zlib 库，然后重新 `cmake .. && make`。

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

如果构建成功，你会在 `build` 文件夹下找到 `gcd_array_generator`, `stu_assign_generator` 等可执行文件。

**运行与测试**

```shell
# 在 build 目录下操作
$ ./gcd_array_generator > gcd_array_generator.ll
$ lli gcd_array_generator.ll
$ echo $?
```

你可以通过观察原来的 C 代码来推断 `echo $?` 应该返回的正确结果，也可以通过 `clang -S -emit-llvm` 编译 `tests/2-ir-gen/warmup/c_cases` 目录下 4 个 C 文件获得 4 个相应的 .ll 文件，再用 `lli` 执行 .ll 文件来获取正确结果。

### 仓库目录结构

与 Light IR 预热实验相关文件如下

```
.
├── ...
├── include
│   ├── common
│   └── lightir/*
└── tests
    ├── ...
    └── 2-ir-gen
        └── warmup
            ├── CMakeLists.txt
            ├── c_cases           <- 需要翻译的 c 代码
            ├── calculator        <- 助教编写的计算器示例
            ├── stu_cpp           <- 学生需要编写的 .ll 代码手动生成器
            ├── stu_ll            <- 学生需要手动编写的 .ll 代码
            └── ta_gcd            <- 助教编写的 .ll 代码手动生成器示例
```

!!! warning

    我们已经创建了 `stu_ll` 目录下的四个空文件，请不要修改文件名！否则希冀平台的测试将会失败。

## 思考题

1. 在 [Light IR 简介](../common/LightIR.md)里，你已经了解了 IR 代码的基本结构，请尝试编写一个有全局变量的 cminus 程序，并用 `clang` 编译生成中间代码，解释全局变量在其中的位置。
2. Light IR 中基本类型 label 在 Light IR C++ 库中是如何用类表示的？
3. Light IR C++ 库中 `Module` 类中对基本类型与组合类型存储的方式是一样的吗？请尝试解释组合类型使用其存储方式的原因。
