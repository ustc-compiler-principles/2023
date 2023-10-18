# 后端环境配置

本次实验需要安装龙芯的交叉编译工具链。这篇文档将帮助你安装好相关工具。

## 交叉编译

我们的后端面向的是龙芯架构，而虚拟机和宿主机都是 x86 架构，不能直接运行龙芯程序。

所谓的交叉编译，就是在一个平台上生成另一个平台上的可执行代码。

具体到我们的实验，我们将在虚拟机上安装龙芯 gcc、qemu 和 gdb，使得我们可以在虚拟机中生成龙芯的可执行文件、模拟运行并且调试。

## 安装

助教准备了[龙芯交叉编译的相关工具](https://rec.ustc.edu.cn/share/d8c57580-669d-11ee-8794-d542ef642531)，以便同学们在本地完成测试与调试。

具体来说有以下四项：

- **loongarch64-clfs-3.0-cross-tools-gcc-glibc.tar.xz**：生成龙芯汇编、可执行文件等
- **qemu-6.2.50.loongarch64.tar.gz**：模拟运行龙芯二进制文件
- **gdb.tar.gz**：调试生成的龙芯程序

- **test-env.tar.gz**：用于测试上述工具是否安装成功

点击上方链接，下载压缩包至虚拟机中。你可以根据自己的喜好选择存放的位置，比如 `~/Downloads`。

??? info "如何在宿主机和虚拟机之间传送文件？"

    如果你在 Windows 系统上阅读本文档，你大概将网盘中的文件下载到了 Windows 系统（宿主机）中。

    如何将文件传递进虚拟机中呢？在 Lab0 中你已经配置了宿主机与虚拟机之间的[ SSH 连接](../lab0/linux.md/#141-ssh)，现在你可以通过 scp 命令进行文件传输：

    ```shell
    $ scp -P 端口号 源文件 目标文件
    ```
    例如将 `gdb.tar.gz` 传送到虚拟机的 `~/Downloads` 目录，具体步骤如下：

    1. 在 Windows 文件资源管理器中找到下载好的文件
    2. 在文件列表的空白处按住 Shift 点击鼠标右键，选择打开 PowerShell
    3. 在弹出的命令行窗口中键入：`scp -P 主机端口 gdb.tar.gz 虚拟机用户名@127.0.0.1:~/Downloads`

现在进入虚拟机中你选择的目录，通过 `ls` 可以看到下载好的压缩包，执行以下解压命令，将交叉编译工具解压到 `/opt` 目录下：

```shell
$ sudo tar xaf loongarch64-clfs-3.0-cross-tools-gcc-glibc.tar.xz -C /opt
$ sudo tar xaf qemu-6.2.50.loongarch64.tar.gz -C /opt
$ sudo tar xaf gdb.tar.gz -C /opt
```

此时解压好的工具分别位于：

- `/opt/cross-tools.gcc_glibc/bin/loongarch64-unknown-linux-gnu-gcc`
- `/opt/qemu/bin/qemu-loongarch64`
- `/opt/gdb/bin/loongarch64-unknown-linux-gnu-gdb`

需要使用绝对路径访问。为了使用方便和 **Lab3 的顺利测评**，我们将上述路径加入到`PATH`中：

```shell
$ echo "export PATH=\$PATH:/opt/cross-tools.gcc_glibc/bin:/opt/gdb/bin:/opt/qemu/bin" >> ~/.bashrc && source ~/.bashrc
```

## 测试

经过上述步骤，你可以使用以下三个命令简单检查是否安装成功：

??? info "检查龙芯 gcc 版本"

    ```shell
    $ loongarch64-unknown-linux-gnu-gcc -v
    Using built-in specs.
    COLLECT_GCC=loongarch64-unknown-linux-gnu-gcc
    COLLECT_LTO_WRAPPER=/opt/cross-tools.gcc_glibc/bin/../libexec/gcc/loongarch64-unknown-linux-gnu/12.0.1/lto-wrapper
    Target: loongarch64-unknown-linux-gnu
    Configured with: ../configure --prefix=/opt/cross-tools --build=x86_64-cross-linux-gnu --host=x86_64-cross-linux-gnu --target=loongarch64-unknown-linux-gnu --enable-__cxa_atexit --enable-threads=posix --enable-libstdcxx-time --enable-checking=release --with-sysroot=/opt/cross-tools/target --enable-default-pie --enable-languages=c,c++,fortran,objc,obj-c++,lto
    Thread model: posix
    Supported LTO compression algorithms: zlib zstd
    gcc version 12.0.1 20220210 (experimental) (GCC)
    ```

??? info "检查龙芯 qemu 版本"

    ```shell
    $ qemu-loongarch64 -version
    qemu-loongarch64 version 8.1.1
    Copyright (c) 2003-2023 Fabrice Bellard and the QEMU Project developers
    ```

??? info "检查龙芯 gdb 版本"

    ```shell
    $ loongarch64-unknown-linux-gnu-gdb -v

    GNU gdb (GDB) 12.0.50.20210713-git
    Copyright (C) 2021 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.```
    ```

更进一步，我们提供了环境测试程序。

进入虚拟机中存放压缩包的目录，解压压缩包 `test-env.tar.gz`：

```shell
$ tar xaf test-env.tar.gz
```

解压得到文件夹 `test-env`，其结构及说明如下：

```shell
test-env
├── bubble-sort.S 		# 冒泡排序的汇编实现
├── hello-world.S 		# 输出 hello world 的汇编实现
├── inline-assembly.c	# 内嵌汇编的 c 程序
└── Makefile 			# 使用 make 自动测试 gcc 和 qemu 是否安装成功
```

### 测试 gcc 与 qemu

进入上述 `test-env` 文件夹中，`make`，三段源程序将被编译成面向龙芯的可执行文件，并用 qemu 模拟运行。

<details>
    <summary>期望输出</summary>
    loongarch64-unknown-linux-gnu-gcc -static hello-world.S -o hello-world</br>
    loongarch64-unknown-linux-gnu-gcc -static bubble-sort.S -o bubble-sort</br>
    loongarch64-unknown-linux-gnu-gcc -static inline-assembly.c -o inline-assembly</br>
    loongarch64-unknown-linux-gnu-gcc -O2 -static inline-assembly.c -o inline-assembly-opt</br>
    qemu-loongarch64 ./hello-world</br>
    Hello World!</br>
    qemu-loongarch64 ./bubble-sort</br>
    53461</br>
    13456</br>
    qemu-loongarch64 ./inline-assembly</br>
    ret_1 ret 123</br>
    myadd_1 = 8</br>
    myadd_2 = 8</br>
    myadd_3 = 8</br>
    a = 2, b = 3, c = 2</br>
    a = 2, b = 3, c = 2</br>
    test5 ok</br>
    test6 expected error</br>
    qemu-loongarch64 ./inline-assembly-opt</br>
    ret_1 ret 123</br>
    myadd_1 = 8</br>
    myadd_2 = 8</br>
    myadd_3 = 8</br>
    a = 2, b = 2, c = 1</br>
    a = 2, b = 3, c = 2</br>
    test5 ok</br>
    test6 expected error</br>
</details>
如果得到以上输出，说明 gcc 和 qemu 被正确安装，你已经可以正常进行 Lab3 的评测。

### 测试 / 使用 gdb

!!! warning "Lazy Reading"

    这一小节是为了调试生成的汇编，你可以在需要调试时再回来查看。

我们接下来调试程序 `test-env/bubble-sort.S`，你现在应该去扫一眼这个文件，注意文件最后，`main` 函数如下：

```asm
.global main
main:
    bl display		# 这是第一条汇编指令，率先被执行，表示将跳转进入 display 函数中
    bl sort
    bl display

    li.w $a7, 93	# exit
    li.w $a0, 0
    syscall 0x0
```

在 [测试 gcc 与 qemu](#测试-gcc-与-qemu) 环节，已经利用龙芯 gcc 生成了可执行文件 `bubble-sort`，我们将读取这个文件进行调试。

龙芯程序运行在 qemu 上，首先需要启动 qemu，再用 gdb 连接。在 `test-env` 目录下：

1. 使用 qemu 模拟运行 `bubble-sort`

   ```shell
   $ qemu-loongarch64 -g 1234 bubble-sort &
   ```

   参数 `-g 1234` 表示不立即执行而是等待 gdb 连接端口 1234，结尾的 `&` 符号令 qemu 程序在后台运行。

2. 启动 gdb 并连接 qemu

   ```shell
   $ loongarch64-unknown-linux-gnu-gdb --quiet

   (gdb)
   ```

   我们使用 `--quiet` 参数避免 gdb 输出冗长的版本信息。出现提示符 `(gdb)`，表示我们处于 gdb 的命令行环境中。

   指定可执行文件：

   ```shell
   (gdb) file bubble-sort
   Reading symbols from bubble-sort...
   ```

   连接 qemu：

   ```shell
   (gdb) target remote localhost:1234
   Remote debugging using localhost:1234
   warning: Can not parse XML target description; XML support was disabled at compile time
   _start () at ../sysdeps/loongarch/start.S:45
   45	../sysdeps/loongarch/start.S: No such file or directory.
   ```

   出现了一些 warning，对我们的调试没有影响，可以忽略之。

3. 现在程序停止在 `_start()` 处，我们让程序停在 `main` 函数的入口处：

   ```shell
   (gdb) break main
   Breakpoint 1 at 0x1200008f4
   (gdb) continue
   Continuing.

   Breakpoint 1, 0x00000001200008f4 in main ()
   ```

   现在，程序停止在 `main` 函数入口处。

4. 将内存中的机器码程序反汇编出来：

   ```shell
   (gdb) disassemble
   Dump of assembler code for function main:
   => 0x00000001200008f4 <+0>:	bl          	-204(0xfffff34)	# 0x120000828 <display>
      0x00000001200008f8 <+4>:	bl          	-100(0xfffff9c)	# 0x120000894 <sort>
      0x00000001200008fc <+8>:	bl          	-212(0xfffff2c)	# 0x120000828 <display>
      0x0000000120000900 <+12>:	ori         	$a7, $zero, 0x5d
      0x0000000120000904 <+16>:	move        	$a0, $zero
      0x0000000120000908 <+20>:	syscall     	0x0
   End of assembler dump.
   ```

   在这里有很多有用的信息，比如你看到了 `main` 函数入口处的全部汇编指令，还有当前正在执行的指令地址是 `0x00000001200008f4`，这也是 `main` 函数的地址……

   通过对比，你会发现反汇编得到的指令与 `bubble-sort.S` 中的手写指令基本一致。

5. 查看当前寄存器状态：

   ```shell
   (gdb) info registers
                     zero               ra               tp               sp
   R0   0000000000000000 00000001200009c4 00000001200947e0 00000040007ffb00
                       a0               a1               a2               a3
   R4   0000000000000001 00000040007ffc58 00000040007ffc68 000000012008c030
                       a4               a5               a6               a7
   R8   0000000000000001 000000012008d060 00000040007ffc50 fffffffffffff000
                       t0               t1               t2               t3
   R12  00000001200008f4 00000040007ffb20 0000000000000000 00000001200932b8
                       t4               t5               t6               t7
   R16  000000012008ee00 000000012008ee00 636a6b6c7c6a2f5b ffffffffffffffff
                       t8                x               fp               s0
   R20  fffffffffefef000 0000000000000000 0000000000000000 0000000120088da8
                       s1               s2               s3               s4
   R24  0000000000000001 00000040007ffc58 0000000000000001 00000001200008f4
                       s5               s6               s7               s8
   R28  00000040007ffc68 0000000000000001 0000000120000468 0000000000000000
   pc             0x1200008f4         0x1200008f4 <main>
   badvaddr       0x0                 0x0
   ```

   你可以重点关注一下 pc 寄存器：其中的值与反汇编得到的 `main` 函数地址一致。

6. 当前程序停止在 `main` 函数入口处，即将执行的是指令 `bl display`，我们执行这条指令，将进入 `display` 函数：

   ```shell
   (gdb) stepi
   0x0000000120000828 in display ()
   ```

更多的实用指令，请善用搜索引擎自助查询，我们也欢迎同学们在群内或者论坛中进行讨论。
