# LightIR 预热

LightIR 预热分两部分：

1. 了解 LLVM IR 与 LightIR 关系，认识 LightIR 的结构及指令语法。在 clang 翻译出的 IR 作为参考下，手动编写 IR 文件。
2. 了解并掌握 LightIR C++ 库的使用方法，通过调用 LightIR C++ 库提供的接口生成 IR 文件。

## 手动编写 IR 文件

### LLVM IR 介绍

根据[维基百科](https://zh.wikipedia.org/zh-cn/LLVM)的介绍，LLVM 是一个自由软件项目，它是一种编译器基础设施，以 C++ 写成，包含一系列模块化的编译器组件和工具链，用来开发编译器前端和后端。IR 的全称是 Intermediate Representation，即中间表示。LLVM IR 是 LLVM 项目定义的编译器中间代码。

LLVM IR 指令参考手册：[Reference Manual](https://llvm.org/docs/LangRef.html)

### LightIR 介绍

LLVM IR 的目标是成为一种通用 IR（支持包括动态与静态语言），因此 IR 指令种类较为复杂繁多。本课程从 LLVM IR 中裁剪出了适用于教学的精简的 IR 子集，并将其命名为 LightIR。

LightIR 指令参考手册：[LightIR 手册](../common/LightIR.md#ir-%E6%A0%BC%E5%BC%8F)

### 利用 clang 生成 LLVM IR

clang 是 LLVM 工具链中提供的编译器，在命令行中使用 `clang -S -emit-llvm <c_file>` 命令（其中<c_file>是 c 文件路径），可以得到对应的 `*.ll` 文件，实现从 C 语言向 LLVM IR 的翻译。

本次实验提供了以下例子：[gcd_array.c](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/tests/2-ir-gen/warmup/ta_gcd/gcd_array.c)。学生可以通过使用 clang 翻译示例，并查阅[LightIR 手册](../common/LightIR.md#ir-%E6%A0%BC%E5%BC%8F)来理解每条 LLVM IR 指令与 c 代码的对应情况。

通过 `lli gcd_array.ll; echo $?` 指令，可以测试 `gcd_array.ll` 执行结果的正确性。其中，

- `lli` 会运行 `*.ll` 文件
- `$?` 的内容是上一条命令所返回的结果，而 `echo $?` 可以将其输出到终端中

在后续实验中会经常用到这两条指令。

### 实验内容

<!-- TODO: 把 2023ustc-jianmu-compiler-ta 换成公开仓库 -->

实验提供了四个简单的 c 程序，分别是 `tests/2-ir-gen/warmup/c_cases/` 目录下的 [assign.c](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/tests/2-ir-gen/warmup/c_cases/assign.c)、 [fun.c](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/tests/2-ir-gen/warmup/c_cases/fun.c)、 [if.c](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/tests/2-ir-gen/warmup/c_cases/if.c) 和 [while.c](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/tests/2-ir-gen/warmup/c_cases/while.c)。你们需要在 `test/2-ir-gen/warmup/stu_ll` 目录中，手工完成自己的 assign_hand.ll、func_hand.ll、if_handf.ll 和 while_hand.ll，以实现与上述四个 C 程序相同的逻辑功能，可以添加必要的注释。`.ll` 文件的注释是以 ";" 开头的。

在手动编写 .ll 文件过程中，可以参考 `clang -S -emit-llvm` 的输出，但是提交的结果必须避免同此输出一字不差。

## 使用 LightIR C++ 库生成 IR 文件

### LightIR C++ 库介绍

LLVM 项目提供了辅助 IR 生成的 C++ 库，但其类继承关系过于复杂，并且存在很多为了编译性能的额外设计，不利于学生理解 IR 抽象。因此实验依据 LLVM 的设计，为 LightIR 提供了配套简化的 C++ 库。与 LLVM IR C++ 库相比，LightIR C++ 库仅保留必要的核心类，简化了核心类的继承关系与成员设计，且给学生提供与 LLVM 相同的生成 IR 的接口。

LightIR IR C++ 库参考手册：[LightIR cpp APIs](../common/LightIR.md#c-apis)

### 使用 LightIR C++ 库生成 IR 示例

仔细阅读实验提供的样例及注释 [gcd_array_generator.cpp](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler-ta/-/blob/master/tests/2-ir-gen/warmup/ta_gcd/gcd_array_generator.cpp)。该 cpp 程序会生成与 gcd_array.c 逻辑相同的 LLVM IR 文件。请结合该例子的注释与[LightIR cpp APIs](../common/LightIR.md#c-apis)，掌握 LightIR C++ 库接口的调用方法。运行该示例的方法请参考[编译、运行](./warmup.md#编译运行)

### 实验内容

实验在 `tests/2-ir-gen/warmup/c_cases/` 目录下提供了四个简单的 c 程序。学生需要在 `tests/2-ir-gen/warmup/stu_cpp/` 目录中，编写 assign_generator.cpp、fun_generator.cpp、if_generator.cpp 和 while_generator.cpp 四个 cpp 程序调用 LightIR 库接口生成与四个 c 程序相同逻辑功能的 IR 文件。

## 实验要求

### 仓库目录结构

与 LightIR 预热实验相关文件如下

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

### 编译、运行

首先将你的实验仓库克隆到本地虚拟机中。要编译和运行中间代码生成器，请按照以下步骤在本地虚拟机中进行操作：

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
$ ./gcd_array_generator
$ lli gcd_array_generator.ll
$ echo $?
```

LightIR 预热实验测试量较少，无批量测试脚本，请对四个 cpp 文件单独测试

## 思考题

1. 在[LightIR 简介](../common/LightIR.md)里，你已经了解了 IR 代码的基本结构，请尝试编写一个有全局变量的 cminus 程序，并用 `clang` 编译生成中间代码，解释全局变量在其中的位置。
2. LightIR 中基本类型 `label` 在 LightIR C++ 库中是如何用类表示的？
3. LightIR C++ 库中 `Module` 类中对基本类型与组合类型存储的方式是一样的吗？请尝试解释组合类型使用其存储方式的原因。
