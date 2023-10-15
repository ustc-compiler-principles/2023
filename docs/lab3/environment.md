# 环境配置

本次实验需要安装龙芯的交叉编译工具链。

## 交叉编译

<!--TODO 说明什么是交叉编译-->

我们面向的是龙芯架构，而我们的虚拟机是 x86 架构，所以在虚拟机上是没有办法直接运行龙芯程序的。在我 x86 机器上配置相关的编译工具链，使我们能够在 x86 机器上编译生成龙芯架构的程序，就是交叉编译。

## 安装

助教准备了[龙芯交叉编译的相关工具](https://rec.ustc.edu.cn/share/d8c57580-669d-11ee-8794-d542ef642531)，以便同学们在本地完成测试与调试。

具体来说有以下四项：

- **loongarch64-clfs-3.0-cross-tools-gcc-glibc.tar.xz**：生成龙芯汇编、二进制可执行文件等。
- **qemu-6.2.50.loongarch64.tar.gz**：模拟龙芯机器，执行二进制文件。
- **gdb.tar.gz**：调试生成的程序。

- **test-env.tar.gz**：用于测试上述工具是否安装成功。

点击上方链接，下载压缩包至虚拟机中。你可以根据自己的喜好选择存放的位置，比如 `~/Downloads`。

现在进入你选择的目录，通过 `ls` 可以看到下载好的压缩包，执行以下解压命令，将交叉编译工具解压到 `/opt` 目录下：

```shell
$ sudo tar xaf loongarch64-clfs-3.0-cross-tools-gcc-glibc.tar.xz -C /opt
$ sudo tar xaf qemu-6.2.50.loongarch64.tar.gz -C /opt
$ sudo tar xaf gdb.tar.gz -C /opt
```

!!! root 用户

    如果你使用 root 用户进行实验，不要使用`sudo`。

此时解压好的工具分别位于：

- `/opt/cross-tools.gcc_glibc/bin/loongarch64-unknown-linux-gnu-gcc`
- `/opt/qemu/bin/qemu-loongarch64`
- `/opt/gdb/bin/loongarch64-unknown-linux-gnu-gdb`

需要使用绝对路径访问。为了使用方便和 **lab3 的顺利测评**，我们将上述路径加入到`PATH`中：

1. 打开 `~/.bashrc`

2. 搜索找到 `export PATH=...` 一行

3. 在这一行**最后追加**：`:/opt/qemu/bin:/opt/cross-tools.gcc_glibc/bin:/opt/gdb/bin`

4. 这一行最后看起来像这样：

   ```shell
   export PATH=$PATH:~/.local/bin:/opt/cross-tools.gcc_glibc/bin:/opt/gdb/bin:/opt/qemu/bin
   ```

## 测试

经过上述步骤正确安装后，你可以使用以下三个命令简单检查是否安装成功：

```shell
$ loongarch64-unknown-linux-gnu-gcc -v
$ qemu-loongarch64 -version
$ loongarch64-unknown-linux-gnu-gdb -v
```

更进一步，我们提供了环境测试程序。

进入[安装](#安装)步骤中选择的下载目录，解压压缩包 `test-env.tar.gz`：

```shell
$ tar xaf test-env.tar.gz
```

此时解压出文件夹 `test-env`，其结构及说明如下：

```shell
test-env
├── bubble-sort.S # 冒泡排序的汇编实现
├── hello-world.S # 输出 hello world 的汇编实现
├── inline-assembly.c # 内嵌汇编的 c 程序
└── Makefile # 使用 make 自动测试 gcc 和 qemu 是否安装成功
```

### 测试 gcc 与 qemu

进入上述`test-env`文件夹中，`make`，三段源程序将被编译成面向龙芯的二进制可执行文件，并用 qemu 模拟运行。如果得到以下输出，说明 gcc 和 qemu 被正确安装：

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
### 测试 gdb

因为程序运行在 qemu 而不是本机上，需要使用 gdb 的 remote 指令连接。

仍然在上述 `test-env` 目录下，使用如下指令启动 qemu，并使用 gdb 连接：

```shell
$ qemu-loongarch64 -g 1234 bubble-sort &
$ loongarch64-unknown-linux-gnu-gdb bubble-sort
# 现在在 gdb 的命令行环境中
# 使用如下命令连接 qemu
(gdb) target remote localhost:1234
# 以下为输出
Remote debugging using localhost:1234
warning: Can not parse XML target description; XML support was disabled at compile time
_start () at ../sysdeps/loongarch/start.S:45
45	../sysdeps/loongarch/start.S: No such file or directory.
```

会出现关于 XML 解析的 warning，对我们没有影响，可以忽略之。

现在程序停止在 `_start()` 处，你可以打断点并执行，比如我们让程序停在 `main` 函数的入口处：

```shell
(gdb) break main
Breakpoint 1 at 0x1200008f4
(gdb) continue
Continuing.

Breakpoint 1, 0x00000001200008f4 in main ()
```

至此，gdb 的测试结束。

#### gdb 使用

继续刚才的示例，我们演示调试程序的一般步骤，方便同学们快速上手。

刚刚程序停止在 `main` 函数入口处，我们使用 `disassemble` 反汇编得到当前汇编段落：

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

你会发现反汇编得到的指令与 `bubble-sort.S` 中的手写指令基本一致。

使用`info registers`检查当前寄存器状态：

```
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

你可以重点关注一下 pc 寄存器：其中的值与反汇编得到的 main 函数地址一致。

继续使用 `stepi`，可以进入 `display` 函数中：

```shell
(gdb) stepi
0x0000000120000828 in display ()
```

更多的实用指令，请善用搜索引擎自助查询，我们也欢迎同学们在群内或者论坛中进行讨论。
