# Lab3 后端代码生成

!!! warning "Deadline"

    **2023 年 11 月 xx 日 23:59**

经过 Lab1 和 Lab2，我们的编译器能够将 Cminusf 源代码翻译成 Light IR。本次实验要求同学们将 IR 翻译成龙芯汇编指令。

## 实验内容

<!--TODO 预估时间-->

### 阶段一

此为预热实验，主要引导同学完成环境的配置并学习相关知识。

1. 查阅[后端环境配置](./environment.md)章节，完成龙芯交叉编译工具链的安装。
2. 阅读[龙芯汇编介绍](../common/asm_intro.md)章节，了解龙芯汇编的基础知识。
3. 阅读[Lab3 实验指导](./guidance.md)的[预热实验](./guidance.md#阶段一：预热实验)章节，简要了解如何使用提供的 C++ 接口生成汇编指令，进行补全汇编的实验并测试。

<!--TODO 手动补全汇编的样例待确定、是否/如何计分待确定-->

阶段一需要完成 `tests/3-codegen/warmup/stu_cpp` 目录下代码的填写。

!!! warning "Deadline"

    **待定**

### 阶段二

来到本次实验的核心：实现编译器后端，即根据 IR 翻译得到龙芯汇编指令。

在进行实验之前，请确保完成了[阶段一](#阶段一)，并且本地仓库已经与上游仓库同步：

```shell
# 在虚拟机实验仓库的路径下输入命令
$ git pull upstream master
```

阅读 [Lab3 实验指导](./guidance.md)，了解实验细节，完成本次实验。

!!! warning "Deadline"

    **待定**

## 实验要求

请将[实验仓库](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler)设置为上游仓库，并获取本次实验更新的内容，具体步骤如下：

将上游仓库设置一个别名（alias），在这里我们用 `upstream`，你也可以改成其他你喜欢的名字。在你 fork 后的本地仓库中：

```shell
$ git remote add upstream https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler.git
```

尝试将远程的更新拉取到本地并进行 merge 操作：

```shell
$ git pull upstream master
```

当 merge 过程出现冲突时，请参考 [Lab0](../lab0/git.md#上下游同步和冲突处理) 来合理的解决冲突。

最后你需要将更改同步到你 fork 得到的远程仓库中：

```shell
$ git push origin master
```

## 提交内容

- 阶段一、二：在希冀平台提交你实验仓库的 url（如 `https://cscourse.ustc.edu.cn/vdir/Gitlab/xxx/2023ustc-jianmu-compiler.git`）。
