# LLVM 等软件环境配置

这部分将会带领同学们初步了解 LLVM、Clang、Git、cmake、Flex、Bison、build-essential，并在我们已经配置好的 Linux 环境下安装相关软件包。

## LLVM、Clang 安装

LLVM 是一个开源软件项目，它是一种编译器基础设施，以 C++写成，包含一系列模块化的编译器组件和工具链。LLVM 对应的文件格式是\*.ll 文件，它的形式是位于高级语言与汇编语言之间的一种中间代码，通常称为 LLVM IR. LLVM IR 使用三地址码，在第一节课上大家已经初步接触；并且 LLVM IR 还将高级语言中的变量名映射为 LLVM IR 形式的名字，这就方便了我们对其进行优化。总之，包括以上两个特性在内的许多 LLVM 特性方便我们对中间代码进行优化，这也是我们选择 LLVM 的原因之一。

Clang 是 LLVM 编译器工具集中用于编译 C、C++、Objective-C 的前端，因此在配置 LLVM 环境时我们同样要下载 Clang。

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

单独安装 gcc、g++、make 等安装包过于繁琐，因此 Ubuntu 提供了一个 build-essential 软件包，该软件包包括了软件编译和构建所需的基本工具和库。它包含了编译 C、C++等语言的编译器、链接器和库文件，以及其他一些常用的构建工具和库文件。

```shell

sudo apt install build-essential

#查看是否成功安装
gcc --version
```

## CMake 安装

CMake 是一个跨平台的编译 (Build) 工具，可以用简单的语句来描述所有平台的编译过程。一个大型项目由多个互相调用的工程共同组成，一些用于生成库文件，一些用于实现逻辑功能，工程之间的调用关系复杂而严格，如果让开发者自己理解文件的编译顺序，需要耗费开发者巨大的精力，因此我们一般通过 makefile 文件来处理这些依赖调用关系（makefile 文件由 make 编译，GNU make 已经内置于 build-essential 包中安装），但因为 makefile 在不同平台的标准不一样，无法有效支持跨平台，因此我们需安装 CMake，程序作者可以编写 CMakeLists.txt 文件，让开发者使用 cmake 编译生成对应平台架构的 makefile 文件，再编译这些文件高效的跨平台快速搭建项目。

想要了解更多 CMake 可参考：[Modern CMake 简体中文版](https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/)。

make 可参考：[make 官方文档英文版](https://www.gnu.org/software/make/manual/make.html)。

make 和 CMake 的区别：[https://zhuanlan.zhihu.com/p/431118510](https://zhuanlan.zhihu.com/p/431118510)。

```shell
# 安装 cmake
sudo apt install cmake
# 检查 cmake 版本是否正确安装
cmake --version
```

## Flex 和 Bison 安装

Flex 是一个生成词法分析器的工具。利用 Flex，我们只需提供词法的正则表达式，就可自动生成对应的 C 代码。

Bison 是一款解析器生成器（parser generator），它可以将 LALR 文法转换成可编译的 C 代码，从而大大减轻程序员手动设计解析器的负担。Bison 是 GNU 对早期 Unix 的 Yacc 工具的一个重新实现，所以文件扩展名为 `.y`（Yacc 的意思是 Yet Another Compiler Compiler）。

**将在 Lab1 对 Flex 和 Bison 作更为详细的介绍，Flex 和 Bison 将作为 Lab1 核心学习内容**

```shell
sudo apt-get install flex bison

# 查看是否成功安装
flex --version
bison --version
```

## 测试

成功安装 clang 与 LLVM 是必要的，在后续实验中我们会经常用到它们。为了测试环境配置是否成功，我们准备了一个简单的程序 Test.c，它的内容如下：

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
    printf("PB%d\n",result);//TODO:若学号前缀不是 PB 可于此修改；
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

它的作用是输出你的学号，请你对其进行修改后执行如下指令：

```
clang -S -emit-llvm Test.c -o Test.ll
lli Test.ll;echo $?
```

正常情况下在执行完第一条指令后文件夹下会出现新生成的 Test.ll 文件。

执行完第二条指令后输出应如下形式：

```
PB20114514	// 你的学号
0			// main 函数返回值
```

请将 Test.ll 保存下来，在 Git 实验中将会使用到该文件。

感兴趣的同学们也可自行阅读生成的 Test.ll 文件，注意观察高级语言中的常量、变量、全局变量、数组和函数等在 LLVM IR 中是如何表示的，在后续的实验中我们会经常和它们打交道。

```shell
cat Test.ll
```
