# LLVM 等软件环境配置

这部分将会带领同学们初步了解 LLVM、Clang、Git、cmake、Flex、Bison、build-essential，并在我们已经配置好的 Linux 环境下安装相关软件包。

## LLVM、Clang 安装

LLVM 是一个用 C++编写的开源编译器基础设施项目，包括模块化编译器组件和工具链。它提供了一种称为 LLVM IR 的中间代码，介于高级语言和汇编语言之间，本课程的实验核心围绕 LLVM 展开。

Clang 是 LLVM 编译器工具集的前端，用于编译 C、C++和 Objective-C。在配置 LLVM 环境时，需要安装 Clang。

想要了解更多 LLVM 相关知识可参考：[LLVM Discussion Forums - Our community includes both users and developers of various compiler technologies within the LLVM project.](https://discourse.llvm.org/)

LLVM 和 Clang 都可通过 Ubuntu 自带的软件包管理器下载，因此我们在配置好的环境下执行以下指令即可下载 LLVM 与 Clang：

```shell
# 更新软件源
sudo apt update
# 更新软件（可选）
sudo apt upgrade
# 安装 LLVM 和 Clang
sudo apt install clang llvm
```

## Git 安装

Git 是一个分布式版本控制软件，目前已经成为最流行的版本管理工具。我们将会经常利用到它的分支合并与分支追踪的功能。

在搭建好的环境下使用如下指令安装 Git：

```shell
sudo apt install git
```

## build-essential 安装

在 Ubuntu 中，为了简化安装过程，你可以使用 build-essential 软件包。它集成了编译 C、C++等语言所需的关键工具和库，包括编译器、链接器以及其他常用构建工具和库。这样，你就不需要单独安装 gcc、g++、make 等包，让编译和构建软件更加便捷。

```shell
sudo apt install build-essential

# 查看是否成功安装
gcc --version
```

## CMake 安装

CMake 是一个用于管理和生成跨平台构建系统的开源工具。它可以帮助开发人员轻松地配置、构建和测试他们的项目，而无需关心目标操作系统或编译器的细节。CMake 的主要目标是使跨平台开发更加容易和可维护，因此它被广泛用于许多开源和商业项目中。

想要了解更多 CMake 可参考：[Modern CMake 简体中文版](https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/)。

扩展阅读：

- make 参考：[make 官方文档英文版](https://www.gnu.org/software/make/manual/make.html)。
- make 和 CMake 的区别：[https://zhuanlan.zhihu.com/p/431118510](https://zhuanlan.zhihu.com/p/431118510)。

```shell
# 安装 cmake
sudo apt install cmake
# 检查 cmake 是否正确安装
cmake --version
```

## Flex 和 Bison 安装

Flex 是一个用于生成词法分析器的工具。它简化了词法分析器的创建过程，只需提供正则表达式描述词法规则，Flex 就能自动生成相应的 C 代码。

Bison 是一款解析器生成器，用于将 LALR 文法转换为可编译的 C 代码，从而显著降低了手动设计解析器的工作量。Bison 是对早期 Unix 上的 Yacc 工具的重新实现，因此通常使用 `.y` 文件扩展名来表示它的语法规则文件（Yacc 是 Yet Another Compiler Compiler 的缩写）。

**我们将在 Lab1 对 Flex 和 Bison 作更为详细的介绍**

```shell
sudo apt-get install flex bison

# 查看是否成功安装
flex --version
bison --version
```

## 测试

为了测试配置是否成功，我们提供了一个简单的测试程序 Test.c，其内容如下：

```c
#include<stdio.h>

int Grade[1];
int Degree[1];
int Number[1];
int grade_mul;
int degree_mul;

void StudentNumber (int u[], int v[], int k[]) {
    int grade;
    int degree;
    int number;
    int result;
    grade = u[0];
    degree = v[0];
    number = k[0];
    result = grade * grade_mul + degree * degree_mul + number;
    //TODO:若学号前缀不是 PB 可于此修改；
    printf("PB%d\n",result);
    return;
}

/**
 * @brief 修改 main 函数中的信息输出你的学号
 *
 * @param Grade[0]:你的年级，如 20，21，22 等
 * @param Degree[0]:你的专业代号，如计科是 11
 * @param Number[0]:你的学生序号，如 0011，4514 等
 * @return int
 */
int main(void) {
    Grade[0] = 20;
    Degree[0] = 11;
    Number[0] = 4514;
    grade_mul = 1000000;
    degree_mul = 10000;
    StudentNumber(Grade,Degree,Number);
    return 0;
}

```

它的作用是输出你的学号，请你填写相关信息后，在虚拟机/服务器中执行如下指令运行：

```shell
# 该指令对Text.c进行编译，并生成其对应的LLVM IR代码，记为Test.ll。
$ clang -S -emit-llvm Test.c -o Test.ll
```

随后，我们使用`lli`执行该 IR 文件：

```shell
$ lli Test.ll && echo $?
PB20114514  # 第一行结果应为你的学号，助教会评测该结果 [重要]
0           # main 函数返回值，应该为0
```

请将你修改的 Test.ll 保存下来，它将在[Git 的使用](git.md)中被使用。

感兴趣的同学们也可自行阅读生成的 Test.ll 文件，注意观察高级语言中的常量、变量、全局变量、数组和函数等在 LLVM IR 中是如何表示的，在后续的实验中我们会经常和它们打交道。

```shell
cat Test.ll
```
