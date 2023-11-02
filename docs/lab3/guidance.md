# Lab3 实验指导

## 阶段一：预热实验

### 仓库目录结构

与预热实验相关的文件如下：

```
.
├── ...
├── include
│   ├── common
│   └── codegen/*
└── tests
    ├── ...
    └── 3-codegen
        └── warmup
            ├── CMakeLists.txt
            ├── ll_cases          <- 需要翻译的 ll 代码
            └── stu_cpp           <- 学生需要编写的汇编代码手动生成器
```

### 实验内容

实验在 `tests/3-codegen/warmup/ll_cases/` 目录下提供了六个 `.ll` 文件。学生需要在 `tests/3-codegen/warmup/stu_cpp/` 目录中，依次完成 `assign_codegen.cpp`、`float_codegen.cpp`、`global_codegen.cpp`、`function_codegen.cpp`、`icmp_codegen.cpp` 和 `fcmp_codegen.cpp` 六个 C++ 程序中的 TODO。这六个程序运行后应该能够生成 `tests/3-codegen/warmup/ll_cases/` 目录下六个 `.ll` 文件对应的汇编程序。

### 编译

```shell
$ cd 2023ustc-jianmu-compiler
$ mkdir build
$ cd build
# 使用 cmake 生成 makefile 等文件
$ cmake ..
# 使用 make 进行编译
$ make
```

如果构建成功，你会在 `build` 文件夹下找到 `stu_assign_codegen`, `stu_float_codegen` 等可执行文件。

### 运行与测试

!!! note

    在运行和测试之前，你应该确保你完成了[后端环境配置](./environment.md)。

```shell
# 在 build 目录下操作
$ ./stu_assign_codegen > assign.s
$ loongarch64-unknown-linux-gnu-gcc -static assign.s -o assign
$ qemu-loongarch64 ./assign
$ echo $?
```

你可以通过观察原来的 `.ll` 代码来推断 `echo $?` 应该返回的正确结果，也可以直接使用 `lli` 执行 `.ll` 文件来获取正确结果。

## 阶段二：编译器后端

### 仓库目录结构

与实验第二阶段相关的文件如下：

```
.
├── include
│   └── codegen/*					# 相关头文件
├── src
│   └── codegen
│       └── CodeGen.cpp     	<-- 学生需要补全的文件
└── tests
    ├── 3-codegen
    │   └── autogen
    │       ├── eval_lab3.sh	<-- 测评脚本
    │		└── testcases 		<-- lab3 第二阶段的测例目录一
    └── testcases_general		<-- lab3 第二阶段的测例目录二
```

### 实验内容

补全 `src/codegen/CodeGen.cpp` 中的 TODO，并按需修改 `include/codegen/CodeGen.hpp` 等文件，使编译器能够生成正确的汇编代码。

!!! info "实现约定"

    出于简化实验的目的，我们只考核最核心的功能点，对如下情况不做要求：
    
    - phi 指令：本实验不考核对于 phi 指令的处理
    
    - 函数调用传参：本次实验只考核使用寄存器传参的情况，即对于超多参数的栈上传参不做要求
    
    此外，存在部分 `void main() {...}` 的样例，其返回值是未定义的，我们要求这样的主函数通过寄存器 `$a0` 返回 0，以顺利进行测评。

### 编译

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

如果构建成功，你会在 `build` 文件夹下找到 cminusfc 可执行文件，它能将 `.cminus` 文件输出为龙芯汇编文件。

!!! note

    为了让 cminusfc 在 `$PATH` 中，一定要 `sudo make install`。

### 运行

完成第二阶段，编译器能够将 `.cminus` 文件翻译成正确的龙芯汇编 `.s` 文件：

```shell
# 假设 cminusfc 的路径在你的 $PATH 中，并且你现在在 test.cminus 文件所在目录中
$ cminusfc test.cminus -S
```

此时会在同目录下生成同名的汇编文件，在这里即为 `test.s`。

为了运行得到的汇编文件，需要调用龙芯交叉编译器编译链接生成二进制文件 `test`：

```shell
# 假设你位于 2023ustc-jianmu-compiler 目录, 否则你应该修改下面 src/io/io.c 到具体的路径
$ loongarch64-unknown-linux-gnu-gcc -static test.s src/io/io.c -o test
```

!!! note

    上面的命令编译了 `test.s`，并链接了 `io.c` 中的函数。

然后你可以使用 `qemu-loongarch64` 运行二进制文件 `test`

```shell
$ qemu-loongarch64 ./test
```

### 测试

在 `tests/3-codegen/autogen` 目录下，使用我们提供的 `eval_lab3.sh` 进行本地评测，其接受两个**参数**：

1. 测试样例目录：考核的两个测例目录分别是 `./testcases` 和 `../../testcases_general`
1. 评测模式：`test` 和 `debug` 两种模式都能正常测评，后者会额外生成 `.ll` 文件

脚本运行后，会留下如下文件/目录来辅助你 **debug**：

- `output/` 目录：保存了评测中编译器产生的文件，如汇编文件、IR 文件和二进制文件等
- `log.txt` 文件：该文件收录了编译器报错信息及评测的比较信息 (`diff` 的结果)

出现错误时请查阅相关文件。

在结束测评后，你可以使用 `./cleanup.sh` **清空**上述文件/目录，以保持仓库的清爽。

以下是评测脚本的**使用示例**：

以 `./testcases` 为测例目录，使用评测脚本 `test` 模式测试（我们使用正确的实现，但是故意在 `13-complex.cminus` 中制造了语法错误）：

```shell
$ ./eval_lab3.sh ./testcases test
[info] Start testing, using testcase dir: ./testcases
0-io...OK
1-return...OK
2-calculate...OK
3-output...OK
4-if...OK
5-while...OK
6-array...OK
7-function...OK
8-store...OK
9-fibonacci...OK
10-float...OK
11-floatcall...OK
12-global...OK
13-complex..../eval_lab3.sh: line 65: 40958 Segmentation fault      (core dumped) bash -c "cminusfc -S $case -o $asm_file" >> $LOG 2>&1
CE: cminusfc compiler error
```

从脚本输出中观察到样例 `13-complex` 没有通过，原因为 `CE: cminusfc compiler error`，表示 `cminusfc` 编译过程报错，打开 `log.txt` 检查报错信息：

```
==========./testcases/0-io.cminus==========
1234							1234
0								0
...省略多行中间输出
==========./testcases/13-complex.cminus==========
error at line 7 column 5: syntax error
```

根据输出信息，我们了解到 `cminusfc` 检测到语法错误而退出，具体为 `13-complex.cminus` 中第 7 行第 5 列。

直接使用初始代码尝试评测，每个样例都会报错 `CE: cminusfc compiler error`。

??? detail "使用初始代码评测后的 log 文件"

    ```
    ==========./testcases/0-io.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_call
    ==========./testcases/1-return.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_call
    ==========./testcases/2-calculate.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/3-output.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_call
    ==========./testcases/4-if.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/5-while.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/6-array.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/7-function.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/8-store.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/9-fibonacci.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/10-float.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/11-floatcall.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ==========./testcases/12-global.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_load
    ==========./testcases/13-complex.cminus==========
    terminate called after throwing an instance of 'not_implemented_error'
    what(): gen_alloca
    ```
