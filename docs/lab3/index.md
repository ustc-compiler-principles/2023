# Lab3 后端代码生成

经过 Lab1 和 Lab2，我们的编译器能够将 Cminusf 源代码翻译成 Light IR。本次实验要求同学们将 IR 翻译成龙芯汇编指令。

## 同步实验仓库

在进行实验之前，首先拉取[实验仓库](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler)的最新代码，具体步骤可以参考[Lab2 中的指导](../lab2/index.md#实验要求)。

本次实验仓库更新的内容如下，每个阶段的文件将在对应文档详细说明：

```
.
├── ...
├── include
│   ├── ...
│   └── codegen
│       ├── ASMInstruction.hpp  # 描述汇编指令
│       ├── CodeGen.hpp         # 后端框架顶层设计
│       ├── CodeGenUtil.hpp     # 一些辅助函数及宏的定义
│       └── Register.hpp        # 描述寄存器
├── src
│   ├── ...
│   └── codegen
│       ├── CMakeLists.txt
│       ├── CodeGen.cpp     <-- lab3 第二阶段需要修改的文件
│       └── Register.cpp
└── tests
    ├── ...
    └── 3-codegen
        ├── warmup			<-- lab3 第一阶段（代码撰写）
        └── autogen			<-- lab3 第二阶段的测试
```

## 实验内容

<!--TODO 预估时间-->

### 阶段一

此为预热实验，主要引导同学完成环境的配置并学习相关知识。

1. 阅读[后端环境配置](./environment.md)章节，完成龙芯交叉编译工具链的安装。
2. 阅读[龙芯汇编介绍](../common/asm_intro.md)章节，了解龙芯汇编的基础知识。
3. 阅读[后端框架介绍](framework.md)中的[指令类介绍](framework.md/#指令类)和[辅助函数（添加指令）介绍](framework.md/#append_inst)，了解如何使用提供的 C++ 接口生成汇编指令。
4. 阅读 [Lab3 实验指导的预热实验](./guidance.md#阶段一预热实验)章节，了解实验内容，进行补全汇编的实验并测试。

!!! warning "Deadline"

    **2023 年 11 月 10 日 23:59**

### 阶段二

来到本次实验的核心：实现编译器后端，即根据 IR 翻译得到龙芯汇编指令。

一个典型的编译器后端从中间代码获取信息，进行活跃变量分析、寄存器分配、指令选择、指令优化等一系列流程，最终生成高质量的后端代码。

本次实验，我们将这些复杂的流程简化，仅追求实现的完整性，要求同学们采用栈式分配的策略，完成后端代码生成。

1. 阅读[栈式分配介绍](stack_allocation.md)章节，了解其思想。
2. 阅读[后端框架介绍](framework.md)章节，了解实验代码框架设计。
3. 阅读 [Lab3 实验指导中的阶段二](./guidance.md/#阶段二编译器后端)章节，了解实验细节，完成本次实验。

!!! warning "Deadline"

    **2023 年 11 月 24 日 23:59**

## 提交内容

阶段一、二：在希冀平台提交你实验仓库的 url（如 `https://cscourse.ustc.edu.cn/vdir/Gitlab/xxx/2023ustc-jianmu-compiler.git`）。

在提交之前，请确保你 fork 得到的远程仓库与本地同步：

```shell
$ git push origin master
```
