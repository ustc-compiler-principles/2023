# Lab3 实验指导

一个典型的编译器后端从中间代码获取信息，进行活跃变量分析、寄存器分配、指令选择、指令优化等一系列流程，最终生成高质量的后端代码。

本次实验，我们将这些复杂的流程简化，仅追求实现的完整性，要求同学们采用栈式分配的策略，完成后端代码生成。

## 栈式分配

阅读[栈式分配介绍](stack_allocation.md)章节，了解其思想。

## 实验框架

阅读[后端框架介绍](framework.md)章节，了解实验代码框架设计。

## 实现约定

出于简化实验的目的，我们只考核最核心的功能点，对如下情况不做要求：

- phi 指令：本实验不考核对于 phi 指令的处理
- 函数调用传参：本次实验只考核使用寄存器传参的情况，即对于超多参数的栈上传参不做要求

此外，存在部分 `void main() {...}` 的样例，其返回值是未定义的，我们要求这样的主函数通过寄存器 `$a0` 返回 0，以顺利进行测评。

## 本地评测

!!! warning "编译"

    评测之前，确保你已经在项目 `build` 目录下进行了 `sudo make install`，使程序 `cminusfc` 是最新版本。详情可以查询[ lab2 的相关章节](../lab2/autogen.md#编译运行和评测)。

### 评测脚本

在 `tests/3-codegen` 目录下，使用我们提供的 `eval_lab3.sh` 进行本地评测，其接受两个**参数**：

1. 测试样例目录：考核的两个测例目录分别是 `./testcases` 和 `../testcases_general`
1. 评测模式：`test` 和 `debug` 两种模式都能正常测评，后者会额外生成 `.ll` 文件

脚本运行后，会留下如下文件/目录来辅助你 **debug**：

- `output/` 目录：保存了评测中编译器产生的文件，如汇编文件、IR 文件和二进制文件等
- `log.txt` 文件：该文件收录了编译器报错信息及评测的比较信息 (`diff` 的结果)

出现错误时请查阅相关文件。

在结束测评后，你可以使用 `./cleanup.sh` **清空**上述文件/目录，以保持仓库的清爽。

### 示例

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

<details>
    <summary>初始代码评测后的 log 文件</summary>
    ==========./testcases/0-io.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_call</br>
    ==========./testcases/1-return.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_call</br>
    ==========./testcases/2-calculate.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/3-output.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_call</br>
    ==========./testcases/4-if.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/5-while.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/6-array.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/7-function.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/8-store.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/9-fibonacci.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/10-float.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/11-floatcall.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
    ==========./testcases/12-global.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_load</br>
    ==========./testcases/13-complex.cminus==========</br>
    terminate called after throwing an instance of 'not_implemented_error'</br>
      what():  gen_alloca</br>
</details>
