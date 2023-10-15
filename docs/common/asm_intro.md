# 龙芯汇编介绍

> 本介绍参考自[《龙芯架构参考手册 - 卷一：基础架构》](https://github.com/loongson/LoongArch-Documentation/releases/latest/download/LoongArch-Vol1-v1.02-CN.pdf)。

龙芯架构 LoongArch 时一种精简指令集计算机（RISC）风格的指令系统架构，分为 32 位和 64 位两个版本，分别称为 LA32 架构和 LA64 架构。我们主要关注 LA64 架构中的非向量整数和非向量浮点指令中的一部分，如果你想了解更多关于龙芯汇编的内容，可以参考[《龙芯架构参考手册 - 卷一：基础架构》](https://github.com/loongson/LoongArch-Documentation/releases/latest/download/LoongArch-Vol1-v1.02-CN.pdf)。

??? info "LA32 和 LA64 之间的关系"

    LA64 架构应用级向下二进制兼容 LA32 架构，即采用 LA32 架构的**应用软件**（不包括操作系统内核等系统软件）的二进制可以直接运行在兼容 LA64 架构的机器上并取得相同的运行结果。

如下是一段 C 语言代码 `easy.c` 与其对应的龙芯汇编 `easy.s` 示例。

- `easy.c`

  ```c
  int main(void) {
    int a;
    int b;
    a = 1;
    b = 2;
    return a + b;
  }
  ```

- `easy.s`

  ```asm
  	.text
  	.globl main
  	.type main, @function
  main:
  	st.d $ra, $sp, -8
  	st.d $fp, $sp, -16
  	addi.d $fp, $sp, 0
  	addi.d $sp, $sp, -64
  .main_label_entry:
  # %op0 = alloca i32
  	addi.d $t0, $fp, -28
  	st.d $t0, $fp, -24
  # %op1 = alloca i32
  	addi.d $t0, $fp, -40
  	st.d $t0, $fp, -36
  # store i32 1, i32* %op0
  	addi.w $t0, $zero, 1
  	ld.d $t1, $fp, -24
  	st.w $t0, $t1, 0
  # store i32 2, i32* %op1
  	addi.w $t0, $zero, 2
  	ld.d $t1, $fp, -36
  	st.w $t0, $t1, 0
  # %op2 = load i32, i32* %op0
  	ld.d $t0, $fp, -24
  	ld.w $t0, $t0, 0
  	st.w $t0, $fp, -44
  # %op3 = load i32, i32* %op1
  	ld.d $t0, $fp, -36
  	ld.w $t0, $t0, 0
  	st.w $t0, $fp, -48
  # %op4 = add i32 %op2, %op3
  	ld.w $t0, $fp, -44
  	ld.w $t1, $fp, -48
  	add.w $t2, $t0, $t1
  	st.w $t2, $fp, -52
  # ret i32 %op4
  	ld.w $a0, $fp, -52
  	b main_exit
  main_exit:
  	addi.d $sp, $sp, 64
  	ld.d $ra, $sp, -8
  	ld.d $fp, $sp, -16
  	jr $ra
  ```

## 汇编助记格式

龙芯架构对指令名和操作数的前、后缀进行了统一考虑，以方便汇编编程人员和编译器开发人员使用，例如 `fadd.s` 指令为“单精度” “浮点”加法指令。

### 指令名前缀

龙芯架构指令前缀和指令类型的对应关系为：

| 指令前缀 |     指令类型     |
| :------: | :--------------: |
|  无前缀  |  非向量整数指令  |
|   `f`    | 非向量浮点数指令 |
|    ……    |        ……        |

### 指令名后缀

其次，绝大多数指令通过指令名中 `.XX` 形式的后缀来指示指令的操作对象类型：

- 对于整数指令，后缀和操作对象类型的对应关系为

  | 指令后缀 | 操作对象类型 |  操作对象长度  |
  | :------: | :----------: | :------------: |
  |   `.b`   |  有符号字节  | 1 字节 / 8 位  |
  |   `.h`   |  有符号半字  | 2 字节 / 16 位 |
  |   `.w`   |   有符号字   | 4 字节 / 32 位 |
  |   `.d`   |  有符号双字  | 8 字节 / 64 位 |
  |  `.bu`   |  无符号字节  | 1 字节 / 8 位  |
  |  `.hu`   |  无符号半字  | 2 字节 / 16 位 |
  |  `.wu`   |   无符号字   | 4 字节 / 32 位 |
  |  `.du`   |  无符号双字  | 8 字节 / 64 位 |

  !!! info "特殊情况"

      当操作数是有符号数还是无符号数不影响运算结果时，指令名中携带的后缀均不含 `u`，但是并不意味着指令的操作对象只能是有符号数。

- 对于浮点数指令，后缀和指令类型的对应关系为

  | 指令后缀 | 操作对象类型 |  操作对象长度  |
  | :------: | :----------: | :------------: |
  |   `.h`   | 半精度浮点数 | 2 字节 / 16 位 |
  |   `.s`   | 单精度浮点数 | 4 字节 / 32 位 |
  |   `.d`   | 双精度浮点数 | 8 字节 / 64 位 |
  |   `.w`   |   有符号字   | 4 字节 / 32 位 |
  |   `.l`   |  有符号双字  | 8 字节 / 64 位 |
  |  `.wu`   |   无符号字   | 4 字节 / 32 位 |
  |  `.lu`   |  无符号双字  | 8 字节 / 64 位 |

当源操作数和目的操作数的数据位宽和有无符号情况一致时，指令名只有一个后缀。但是如果源操作数的数据位宽和有无符号情况一致，但和目的操作数不一致时，那么指令名将有两个后缀：

- 左边的后缀表示目的操作数的情况
- 右边的操作数表示源操作数的情况

比如，将长整数型定点数（有符号双字）转化为单精度浮点数的指令名为 `ffint.s.l`。

更复杂的数据位宽情况这里不做讨论，感兴趣的同学可以自行查阅[《龙芯架构参考手册 - 卷一：基础架构》](https://github.com/loongson/LoongArch-Documentation/releases/latest/download/LoongArch-Vol1-v1.02-CN.pdf)的 1.3 章节。

### 寄存器首字母

寄存器操作数通过不同的首字母表明其属于哪个寄存器文件。对于非向量寄存器，以 `$rN` 标记通用寄存器，`$fN` 标记浮点寄存器。其中 `N` 是数字，表示操作的时该寄存器文件中的第 N 号寄存器。

## 寄存器

### 通用寄存器

整数指令涉及的寄存器包括**通用寄存器** GR 和**程序计数器** PC。

其中通用寄存器有 32 个，记为 `$r0` ~ `$r31`，其中第 0 号寄存器 `$r0` 的值恒为 `0`。在 LA64 架构下，通用寄存器的位宽为 64 比特。在龙芯架构 ELF psABI 规范中，通用寄存器遵循如下的使用规定：

|      名称       |     别名      |           用途           | 在调用中是否保留 |
| :-------------: | :-----------: | :----------------------: | :--------------: |
|      `$r0`      |    `$zero`    |         常数 `0`         |     （常数）     |
|      `$r1`      |     `$ra`     |         返回地址         |        否        |
|      `$r2`      |     `$tp`     |         线程指针         |   （不可分配）   |
|      `$r3`      |     `$sp`     |          栈指针          |        是        |
|  `$r4` - `$r5`  | `$a0` - `$a1` | 传参寄存器、返回值寄存器 |        否        |
| `$r6` - `$r11`  | `$a2` - `$a7` |        传参寄存器        |        否        |
| `$r12` - `$r20` | `$t0` - `$t8` |        临时寄存器        |        否        |
|     `$r21`      |               |           保留           |   （不可分配）   |
|     `$r22`      | `$fp` / `$s9` |  栈帧指针 / 静态寄存器   |        是        |
| `$r23` - `$r31` | `$s0` - `$s8` |        静态寄存器        |        是        |

如果“在调用中是否保留”标志为“是”，则表示在调用函数后，寄存器中的值不会发生更改，这意味着被调用的函数需要确保在返回之前还原这些寄存器的值。

### 浮点寄存器

浮点数指令涉及到的寄存器包括**浮点寄存器** FR、**条件标志寄存器** CFR 和**浮点控制状态寄存器** FCSR。我们将着重介绍前两种寄存器。

#### 浮点寄存器

浮点寄存器共有 32 个，记为 `$f0` ~ `$f31`，每一个都可以读写。在龙芯架构 ELF psABI 规范中，浮点寄存器遵循如下的使用规定：

|      名称       |       别名       |           用途           | 在调用中是否保留 |
| :-------------: | :--------------: | :----------------------: | :--------------: |
|  `$f0` - `$f1`  | `$fa0` - `$fa1`  | 传参寄存器、返回值寄存器 |        否        |
|  `$f2` - `$f7`  | `$fa2` - `$fa7`  |        传参寄存器        |        否        |
| `$f8` - `$f23`  | `$ft0` - `$ft15` |        临时寄存器        |        否        |
| `$f24` - `$f31` | `$fs0` - `$fs7`  |        静态寄存器        |        是        |

如果“在调用中是否保留”标志为“是”，则表示在调用函数后，寄存器中的值不会发生更改，这意味着被调用的函数需要确保在返回之前还原这些寄存器的值。

#### 条件标志寄存器

条件标志寄存器共有 8 个，记为 `$fcc0` ~ `$fcc7`，每一个都可以读写，位宽为 1 比特。

- **浮点比较**的结果将写到条件标志寄存器中，比较结果为真时置 `1`，否则置 `0`。
- **浮点分支指令的判断条件**来自条件标志寄存器。

## 整数指令

在龙芯架构中，基础整数指令操作的数据类型有 5 种：

| 数据类型 |    英文    | 简记 | 长度 |
| :------: | :--------: | :--: | :--: |
|   比特   |    bit     |  b   |  1b  |
|   字节   |    Byte    |  B   |  8b  |
|   半字   |  Halfword  |  H   | 16b  |
|    字    |    Word    |  W   | 32b  |
|   双字   | Doubleword |  D   | 64b  |

字节、半字、字和双字数据类型均采用二进制补码的编码方式。

<!-- TODO 整数指令 -->

### 算数运算指令

#### add.w, add.d, sub.w, sub.d

- `add.w` / `sub.w`

  - 指令格式：`add.w $rd, $rj, $rk` / `sub.w $rd, $rj, $rk`

  - 行为：将通用寄存器 `$rj` 中的 `[31:0]` 位数据加上/减去通用寄存器 `$rk` 中的 `[31:0]` 位数据，所得结果的 `[31:0]` 位符号扩展后写入通用寄存器 `$rd` 中

    ```c
    // add.w $rd, $rj, $rk
    tmp = GR[rj][31:0] + GR[rk][31:0]
    GR[rd] = SignExtend(tmp[31:0], 64)
    // sub.w $rd, $rj, $rk
    tmp = GR[rj][31:0] - GR[rk][31:0]
    GR[rd] = SignExtend(tmp[31:0], 64)
    ```

- `add.d` / `sub.d`

  - 指令格式：`add.d $rd, $rj, $rk` / `sub.d $rd, $rj, $rk`

  - 行为：将通用寄存器 `$rj` 中的 `[63:0]` 位数据加上/减去通用寄存器 `$rk` 中的 `[63:0]` 位数据，所得结果写入通用寄存器 `$rd` 中

    ```c
    // add.d $rd, $rj, $rk
    tmp = GR[rj][63:0] + GR[rk][63:0]
    GR[rd] = tmp[63:0]
    // sub.d $rd, $rj, $rk
    tmp = GR[rj][63:0] - GR[rk][63:0]
    GR[rd] = tmp[63:0]
    ```

#### mul.w, mul.d, div.w, div.d

- `mul.w` / `div.w`

  - 指令格式：`mul.w $rd, $rj, $rk` / `div.w $rd, $rj, $rk`

  - 行为：将通用寄存器 `$rj` 中的 `[31:0]` 位数据乘以/除去通用寄存器 `$rk` 中的 `[31:0]` 位数据，所得乘积/商的 `[31:0]` 位数据符号扩展后写入通用寄存器 `$rd` 中

    ```c
    // mul.w $rd, $rj, $rk
    product = signed(GR[rj][31:0]) * signed(GR[rk][31:0])
    GR[rd] = SignExtend(product[31:0], 64)
    // div.w $rd, $rj, $rk
    quotient = signed(GR[rj][31:0]) / signed(GR[rk][31:0])
    GR[rd] = SignExtend(quotient[31:0], 64)
    ```

- `mul.d` / `div.d`

  - 指令格式：`mul.d $rd, $rj, $rk` / `div.d $rd, $rj, $rk`

  - 行为：将通用寄存器 `$rj` 中的 `[63:0]` 位数据乘以/除去通用寄存器 `$rk` 中的 `[63:0]` 位数据，所得乘积/商的 `[63:0]` 位数据写入通用寄存器 `$rd` 中

    ```c
    // mul.d $rd, $rj, $rk
    product = signed(GR[rj][63:0]) * signed(GR[rk][63:0])
    GR[rd] = product[63:0]
    // div.d $rd, $rj, $rk
    quotient = signed(GR[rj][63:0]) / signed(GR[rk][63:0])
    GR[rd] = quotient[63:0]
    ```

#### addi.w, addi.d

- `addi.w`

  - 指令格式：`addi.w $rd, $rj, si12`

  - 行为：将通用寄存器 `$rj` 中的 `[31:0]` 位数据加上 12 比特立即数 `si12` 符号扩展后的 32 位数据，所得结果的 `[31:0]` 位符号扩展后写入通用寄存器 `$rd` 中

    ```c
    tmp = GR[rj][31:0] + SignExtend(si12, 32)
    GR[rd] = SignExtend(tmp[31:0], 64)
    ```

- `addi.d`

  - 指令格式：`addi.d $rd, $rj, si12`

  - 行为：将通用寄存器 `$rj` 中的 `[63:0]` 位数据加上 12 比特立即数 `si12` 符号扩展后的 64 位数据，所得结果写入通用寄存器 `$rd` 中

    ```c
    tmp = GR[rj][63:0] + SignExtend(si12, 64)
    GR[rd] = tmp[63:0]
    ```

#### addu16i.d

- `addu16i.d`

  - 指令格式：`addu16i.d $rd, $rj, si16`

  - 行为：将 16 比特立即数 `si16` 逻辑左移 16 位后再符号扩展，所得数据加上通用寄存器 `$rj` 中的 `[63:0]` 位数据，相加结果写入通用寄存器 `$rd` 中

    ```c
    tmp = GR[rj][63:0] + SignExtend({ si16, 16'b0 }, 64)
    GR[rd] = tmp[63:0]
    ```

#### lu12i.w, lu32i.d, lu52i.d

- `lu12i.w`

  - 指令格式： `lu12i.w $rd, si20`

  - 行为：将 20 比特立即数 `si20` 后的数据最低位连接上 12 比特 0（即 `si20` 逻辑左移 12 位），然后符号扩展后写入通用寄存器 `$rd` 中

    ```c
    GR[rd] = SignExtend({ si20, 12'b0 }, 64)
    ```

  - 例子：`lu12i.w $t0, 0x12345` 执行后，通用寄存器 `$t0` 的值为 `0x0000_0000_1234_5000`

- `lu32i.d`

  - 指令格式：`lu32i.d $rd, si20`

  - 行为：将 20 比特立即数 `si20` 符号扩展后的数据最低位连接上通用寄存器 `$rd` 中 `[31:0]` 位数据，结果写入到通用寄存器 `$rd` 中

    ```c
    GR[rd] = { SignExtend(si20, 32), GR[rd][31:0] }
    ```

  - 例子：设通用寄存器 `$t0` 中的值为 `0x0123_4567_89AB_CDEF`，在 `lu32i.d $t0, 0x12345` 执行后，`$t0` 中的值就变成了 `0x0001_2345_89AB_CDEF`

- `lu52i.d`

  - 指令格式： `lu52i.d $rd, $rj, si12`

  - 行为：将 12 比特立即数 `si12`最低位连接上通用寄存器 `$rj` 中 `[51:0]` 位数据，结果写入到通用寄存器 `$rd` 中

    ```c
    GR[rd] = { si12, GR[rj][51:0] }
    ```

  - 例子：设通用寄存器 `$t0` 中的值为 `0x0123_4567_89AB_CDEF`，在 `lu32i.d $t1, $t0, 0xABC` 执行后，`$t1` 中的值为 `0xABC3_4567_89AB_CDEF`

### 整数比较指令

#### slt, sltu

- `slt`

  - 指令格式：`slt $rd, $rj, $rk`

  - 行为：将通用寄存器 `$rj` 中的数据与通用寄存器 `$rk` 中的数据视**作有符号整数**进行大小比较，如果前者小于后者，则将通用寄存器 `$rd` 的值置为 `1`，否则置为 `0`

    ```c
    GR[rd] = (signed(GR[rj]) < signed(GR[rk])) ? 1 : 0
    ```

- `sltu`

  - 指令格式：`sltu $rd, $rj, $rk`

  - 行为：将通用寄存器 `$rj` 中的数据与通用寄存器 `$rk` 中的数据**视作无符号整数**进行大小比较，如果前者小于后者，则将通用寄存器 `$rd` 的值置为 `1`，否则置为 `0`

    ```c
    GR[rd] = (unsigned(GR[rj]) < unsigned(GR[rk])) ? 1 : 0
    ```

#### slti, sltui

- `slti`

  - 指令格式：`slti $rd, $rj, si12`

  - 行为：将通用寄存器 `$rj` 中数据与 12 比特立即数 `si12` 符号扩展后所得的数据**视作有符号整数**进行大小比较，如果前者小于后者，则将通用寄存器 `$rd` 的值置为 `1`，否则置为 `0`

    ```c
    tmp = SignExtend(si12, 64)
    GR[rd] = (signed(GR[rj]) < signed(tmp)) ? 1 : 0
    ```

- `sltui`

  - 指令格式：`sltui $rd, $rj, si12`

  - 行为：将通用寄存器 `$rj` 中数据与 12 比特立即数 `si12` 符号扩展后所得的数据**视作无符号整数**进行大小比较，如果前者小于后者，则将通用寄存器 `$rd` 的值置为 `1`，否则置为 `0`

    ```c
    tmp = SignExtend(si12, 64)
    GR[rd] = (unsigned(GR[rj]) < unsigned(tmp)) ? 1 : 0
    ```

### 位运算指令

#### and, or, nor, xor

- 指令格式：

  ```asm
  and $rd, $rj, $rk
  or  $rd, $rj, $rk
  nor $rd, $rj, $rk
  xor $rd, $rj, $rk
  ```

- 行为：将通用寄存器 `$rj` 中的数据与通用寄存器 `$rk` 中的数据进行按位逻辑与/或/或非/异或运算，结果写入通用寄存器 `$rd` 中

  ```c
  // and $rd, $rj, $rk
  GR[rd] = GR[rj] & GR[rk]
  // or  $rd, $rj, $rk
  GR[rd] = GR[rj] | GR[rk]
  // nor $rd, $rj, $rk
  GR[rd] = ~(GR[rj] | GR[rk])
  // xor $rd, $rj, $rk
  GR[rd] = GR[rj] ^ GR[rk]
  ```

#### andn, orn

- 指令格式：

  ```asm
  andn $rd, $rj, $rk
  orn  $rd, $rj, $rk
  ```

- 行为：将通用寄存器 `$rj` 中的数据与通用寄存器 `$rk` 中的数据**按位取反后**的数据进行按位逻辑与/或运算，结果写入通用寄存器 `$rd` 中

  ```c
  // andn $rd, $rj, $rk
  GR[rd] = GR[rj] & (~GR[rk])
  // orn  $rd, $rj, $rk
  GR[rd] = GR[rj] | (~GR[rk])
  ```

#### andi, ori, xori

- 指令格式：

  ```asm
  and $rd, $rj, ui12
  or  $rd, $rj, ui12
  xor $rd, $rj, ui12
  ```

- 行为：将通用寄存器 `$rj` 中数据与 12 比特立即数零扩展后的数据进行安慰逻辑与/或/异或运算，结果写入通用寄存器 `$rd` 中

  ```c
  // andi $rd, $rj, ui12
  GR[rd] = GR[rj] & ZeroExtend(ui12, 64)
  // ori  $rd, $rj, ui12
  GR[rd] = GR[rj] | ZeroExtend(ui12, 64)
  // xori $rd, $rj, ui12
  GR[rd] = GR[rj] ^ ZeroExtend(ui12, 64)
  ```

### 位操作指令

#### bstrpick.w, bstrpick.d

- `bstrpick.w`

  - 指令格式：`bstrpick.w $rd, $rj, msbw, lsbw`

  - 行为：提取通用寄存器 `$rj` 中 `[msbw:lsbw]` 位零扩展至 32 位，所形成 32 位中间结果符号扩展后写入通用寄存器 `$rd` 中

    ```c
    bstr32 = ZeroExtend(GR[rj][msbw:lsbw], 32)
    GR[rd] = SignExtend(bstr32[31:0], 64)
    ```

- `bstrpick.d`

  - 指令格式：`bstrpick.d $rd, $rj, msbd, lsbd`

  - 行为：提取通用寄存器 `$rj` 中 `[msbd:lsbd]` 位零扩展至 64 位写入通用寄存器 `$rd` 中

    ```c
    GR[rd] = ZeroExtend(GR[rj][msbd:lsbd], 64)
    ```

### 分支指令

!!! info "指令对齐"

    在龙芯架构中，所有指令均采用 32 位固定长度，且指令的地址都要求 4 字节边界对齐。当指令地址不对齐时将触发地址错例外。

!!! info "地址偏移量"

    对于下面的分支指令，如果在汇编代码中采用直接填入偏移值的方式（而不是填入 `.label`），则汇编表示中的立即数应该填入以字节为单位的偏移值，即指令格式中的 `offs << 2`。
    比如，对于下面的汇编指令
    ```asm
    .label_1:
        b .label_2
    .label_2:
        add $zero, $zero, $zero
    ```
    它等价于
    ```asm
    .label_1:
        b 0x4
    .label_2:
        add $zero, $zero, $zero
    ```

#### beq, bne, blt, bge, bltu, bgeu

这六条分支指令的跳转目标地址计算方式是将指令码中的 16 比特立即数 `offs16` 逻辑左移 2 位后再符号扩展，所得偏移值加上该分支指令的 `PC`。

- `beq` / `bne`

  - 指令格式：`beq $rj, $rd, offs16` / `bne $rj, $rd, offs16`

  - 行为：将通用寄存器 `$rj` 和通用寄存器 `$rd` 的值进行比较，如果两者相等/不等则跳转到目标地址，否则不跳转

    ```c
    // beq $rj, $rd, offs16
    if GR[rj] == GR[rd]:
        PC = PC + SignExtend({ offs16, 2'b0 }, 64)
    // bne $rj, $rd, offs16
    if GR[rj] != GR[rd]:
        PC = PC + SignExtend({ offs16, 2'b0 }, 64)
    ```

- `blt` / `bge`

  - 指令格式：`blt $rj, $rd, offs16` / `bge $rj, $rd, offs16`

  - 行为：将通用寄存器 `$rj` 和通用寄存器 `$rd` 的值视作**有符号数**进行比较，如果前者小于/大于等于后者则跳转到目标地址，否则不跳转

    ```c
    // blt $rj, $rd, offs16
    if signed(GR[rj]) < signed(GR[rd]):
        PC = PC + SignExtend({ offs16, 2'b0 }, 64)
    // bge $rj, $rd, offs16
    if signed(GR[rj]) >= signed(GR[rd]):
        PC = PC + SignExtend({ offs16, 2'b0 }, 64)
    ```

- `bltu` / `bgeu`

  - 指令格式：`bltu $rj, $rd, offs16` / `bgeu $rj, $rd, offs16`

  - 行为：将通用寄存器 `$rj` 和通用寄存器 `$rd` 的值视作**无符号数**进行比较，如果前者小于/大于等于后者则跳转到目标地址，否则不跳转

    ```c
    // blt $rj, $rd, offs16
    if unsigned(GR[rj]) < unsigned(GR[rd]):
        PC = PC + SignExtend({ offs16, 2'b0 }, 64)
    // bge $rj, $rd, offs16
    if unsigned(GR[rj]) >= unsigned(GR[rd]):
        PC = PC + SignExtend({ offs16, 2'b0 }, 64)
    ```

#### beqz, bnez

- 指令格式：

  ```asm
  beqz $rj, offs21
  bnez $rj, offs21
  ```

- 目标地址计算方式：将指令码中的 21 比特立即数 `offs21` 逻辑左移 2 位后再符号扩展，所得偏移值加上该分支指令的 `PC`

- 行为：对通用寄存器 `$rj` 的值进行判断，如果等于/不等于 0 则跳转到目标地址，否则不跳转

  ```c
  // beqz $rj, offs21
  if GR[rj] == 0:
      PC = PC + SignExtend({ offs21, 2'b0 }, 64)
  // bnez $rj, offs21
  if GR[rj] != 0:
      PC = PC + SignExtend({ offs21, 2'b0 }, 64)
  ```

#### b

- 指令格式：`b offs26`

- 目标地址计算方式：将指令码中的 26 比特立即数 `offs26` 逻辑左移 2 位后再符号扩展，所得偏移值加上该分支指令的 `PC`

- 行为：无条件跳转到目标地址处

  ```c
  PC = PC + SignExtend({ offs26, 2'b0 }, 64)
  ```

#### bl

- 指令格式：`bl offs26`

- 目标地址计算方式：将指令码中的 26 比特立即数 `offs26` 逻辑左移 2 位后再符号扩展，所得偏移值加上该分支指令的 `PC`

- 行为：无条件跳转到目标地址处，同时将该指令的 `PC` 值加 4 的结果写入到 1 号通用寄存器 `$r1`（`$ra`）中

  ```c
  GR[1] = PC + 4
  PC = PC + SignExtend({ offs26, 2'b0 }, 64)
  ```

#### jirl

- 指令格式：`jirl $rd, $rj, offs16`

- 目标地址计算方式：将指令码中的 16 比特立即数 `offs16` 逻辑左移 2 位后再符号扩展，所得偏移值加上通用寄存器 `$rj` 中的值

- 行为：无条件跳转到目标地址处，同时将该指令的 `PC` 值加 4 的结果写入到通用寄存器中。

  ```c
  GR[rd] = PC + 4
  PC = GR[rj] + SignExtend({ offs16, 2'b0 }, 64)
  ```

!!! info "`jirl` 指令别名"

    - `jirl $zero, $rj, 0` 指令可以简化为 `jr $rj`。
    - `jirl $zero, $ra, 0`（`jr $ra`）指令常作为调用返回跳转指令。

### 普通访存指令

!!! info "小端序"

    龙芯架构只采用小端序（little endian）的储存方式，即一个字 `0x0123_4567` 在内存中的存储形式如下：

    | 地址 | 字节 |
    |:----:|:----:|
    | `base + 3` | `0x01` |
    | `base + 2` | `0x23` |
    | `base + 1` | `0x45` |
    | `base` | `0x67` |

在介绍普通访存指令时，由于我们主要关注逻辑地址，因此会先不考虑逻辑地址和物理地址之间的转换，尽管这一转换在每条访存指令的执行过程中都会发生。

为了简化实验，可以假设运算环境支持并允许非对齐访存，即 `ld` 和 `st` 指令不需要考虑地址对齐的问题。

#### ld.\*

`ld` 指令可以使用的后缀有 `.b`、`.h`、`.w`、`.d`、`.bu`、`.hu` 和 `.wu`，其中 `b` / `h` / `w` / `d` 分别代表字节/半字/字/双字数据类型，长度分别为 1 / 2 / 4 / 8 字节。

- `ld.b` / `ld.h` / `ld.w`

  - 指令格式：

    ```asm
    ld.b $rd, $rj, si12
    ld.h $rd, $rj, si12
    ld.w $rd, $rj, si12
    ```

  - 访存地址：通用寄存器 `$rj` 中的值与符号扩展后的 12 比特立即数 `si12` 相加求和的结果

  - 行为：从内存取回一个字节/半字/字的数据**符号扩展**后写入通用寄存器 `$rd`

    ```c
    addr = GR[rj] + SignExtend(si12, 64) // 忽略了物理地址转换
    value = LoadMemory(addr, TYPE) // TYPE 分别为 BYTE / HALFWORD / WORD
    GR[rd] = SignExtend(value, 64)
    ```

- `ld.bu` / `ld.hu` / `ld.wu`

  - 指令格式：

    ```asm
    ld.bu $rd, $rj, si12
    ld.hu $rd, $rj, si12
    ld.wu $rd, $rj, si12
    ```

  - 访存地址：通用寄存器 `$rj` 中的值与符号扩展后的 12 比特立即数 `si12` 相加求和的结果

  - 行为：从内存取回一个字节/半字/字的数据**零扩展**后写入通用寄存器 `$rd`

    ```c
    addr = GR[rj] + SignExtend(si12, 64) // 忽略了物理地址转换
    value = LoadMemory(addr, TYPE) // TYPE 分别为 BYTE / HALFWORD / WORD
    GR[rd] = ZeroExtend(value, 64)
    ```

- `ld.d`

  - 指令格式：`ld.d $rd, $rj, si12`

  - 访存地址：通用寄存器 `$rj` 中的值与零扩展后的 12 比特立即数 `si12` 相加求和的结果

  - 行为：从内存取回一个双字的数据写入通用寄存器 `$rd`

    ```c
    addr = GR[rj] + SignExtend(si12, 64) // 忽略了物理地址转换
    GR[rd] = LoadMemory(addr, DOUBLEWORD)
    ```

#### st.\*

`st` 指令可以使用的后缀有 `.b`、`.h`、`.w`、`.d`，分别代表字节/半字/字/双字数据类型，长度分别为 1 / 2 / 4 / 8 字节。

- 指令格式：

  ```asm
  st.b $rd, $rj, si12
  st.h $rd, $rj, si12
  st.w $rd, $rj, si12
  st.d $rd, $rj, si12
  ```

- 访存地址：通用寄存器 `$rj` 中的值与零扩展后的 12 比特立即数 `si12` 相加求和的结果

- 行为：将通用寄存器 `$rd` 中的 `[7:0]` / `[15:0]` / `[31:0]` / `[63:0]` 位数据写入到内存中

  ```c
  // 以 st.w $rd, $rj, si12 为例子
  addr = GR[rj] + SignExtend(si12, 64) // 忽略了物理地址转换
  StoreMemory(GR[rd][31:0], addr, WORD)
  ```

## 浮点数指令

在龙芯架构中，浮点数据类型包括单精度浮点数和双精度浮点数，二者均遵循 IEEE 754-2008 标准规范中的定义。部分浮点指令（如浮点转换指令）也会涉及定点（整数）数据，包括字（Word，简记 W，长度 32b）和长字（Longword，简记 L，长度 64b），它们均采用二进制补码的编码方式。

### 浮点运算指令

#### fadd.s, fadd.d, fsub.s, fsub.d

- `fadd.s` / `fsub.s`

  - 指令格式：`fadd.s $fd, $fj, $fk` / `fsub.s $fd, $fj, $fk`

  - 行为：将浮点寄存器 `$fj` 中的单精度浮点数加上/减去浮点寄存器 `$fk` 中的单精度浮点数，得到的单精度浮点数结果写入到浮点寄存器 `$fd` 中，<u>注意浮点寄存器 `$fd` 中的高 32 位可以是任意值</u>

    ```c
    // fadd.s $fd, $fj, $fk
    FR[fd][31:0] = FP32_add(FR[fj][31:0], FR[fk][31:0])
    // fsub.s $fd, $fj, $fk
    FR[fd][31:0] = FP32_sub(FR[fj][31:0], FR[fk][31:0])
    ```

- `fadd.d` / `fsub.d`

  - 指令格式：`fadd.d $fd, $fj, $fk` / `fsub.d $fd, $fj, $fk`

  - 行为：将浮点寄存器 `$fj` 中的双精度浮点数加上/减去浮点寄存器 `$fk` 中的双精度浮点数，得到的双精度浮点数结果写入到浮点寄存器 `$fd` 中

    ```c
    // fadd.d $fd, $fj, $fk
    FR[fd] = FP64_add(FR[fj], FR[fk])
    // fsub.d $fd, $fj, $fk
    FR[fd] = FP64_sub(FR[fj], FR[fk])
    ```

#### fmul.s, fmul.d, fdiv.s, fdiv.d

- `fmul.s` / `fdiv.s`

  - 指令格式：`fmul.s $fd, $fj, $fk` / `fdiv.s $fd, $fj, $fk`

  - 行为：将浮点寄存器 `$fj` 中的单精度浮点数乘以/除以浮点寄存器 `$fk` 中的单精度浮点数，得到的单精度浮点数结果写入到浮点寄存器 `$fd` 中，<u>注意浮点寄存器 `$fd` 中的高 32 位可以是任意值</u>

    ```c
    // fmul.s $fd, $fj, $fk
    FR[fd][31:0] = FP32_mul(FR[fj][31:0], FR[fk][31:0])
    // fdiv.s $fd, $fj, $fk
    FR[fd][31:0] = FP32_div(FR[fj][31:0], FR[fk][31:0])
    ```

- `fmul.d` / `fdiv.d`

  - 指令格式：`fmul.d $fd, $fj, $fk` / `fdiv.d $fd, $fj, $fk`

  - 行为：将浮点寄存器 `$fj` 中的双精度浮点数乘以/除以浮点寄存器 `$fk` 中的双精度浮点数，得到的双精度浮点数结果写入到浮点寄存器 `$fd` 中

    ```c
    // fmul.d $fd, $fj, $fk
    FR[fd] = FP64_mul(FR[fj], FR[fk])
    // fdiv.d $fd, $fj, $fk
    FR[fd] = FP64_div(FR[fj], FR[fk])
    ```

### 浮点转换指令

#### ffint.\*.\*

`ffint` 指令有两个后缀，其中第一个后缀可以使用 `.s` 和 `.d`，表示转换的目标为单精度/双精度浮点数；第二个后缀可以使用 `.w` 和 `.l`，表示转换的来源是整数型（字，4 字节）/长整数型（双字，8 字节）定点数。

- 指令格式：

  ```asm
  ffint.s.w $fd, $fj
  ffint.s.l $fd, $fj
  ffint.d.w $fd, $fj
  ffint.d.l $fd, $fj
  ```

- 行为：选择**浮点寄存器** `$fj` 中的整数型/长整数型定点数转换为单精度/双精度浮点数，得到的单精度/双精度浮点数写入到浮点寄存器 `$fd` 中

  ```c
  // ffint.s.w $fd, $fj
  FR[fd][31:0] = FP32_convertFromInt(FR[fj][31:0], SINT32)
  // ffint.s.l $fd, $fj
  FR[fd][31:0] = FP32_convertFromInt(FR[fj], SINT64)
  // ffint.d.w $fd, $fj
  FP[fd] = FP64_convertFromInt(FR[fj][31:0], SINT32)
  // ffint.d.l $fd, $fj
  FP[fd] = FP64_convertFromInt(FR[fj], SINT64)
  ```

#### ftintrz.\*.\*

!!! info "舍入模式"

    为了和 LLVM IR/Light IR 中 `fptosi` 的行为一致，我们选择 “向零方向舍入”（rounding towards zero）作为舍入模式，即选择 `ftintrz` 作为转换浮点数为定点数的指令。
    龙芯架构中也有支持其他的舍入模式的 `ftint*` 指令，感兴趣的同学可以参考[《龙芯架构参考手册 - 卷一：基础架构》](https://github.com/loongson/LoongArch-Documentation/releases/latest/download/LoongArch-Vol1-v1.02-CN.pdf)的第 3.2.3.2 节。

`ftintrz` 指令有两个后缀，其中第一个后缀可以使用 `.w` 和 `.l`，表示转换的目标是整数型（字，4 字节）/长整数型（双字，8 字节）定点数；第二个后缀可以使用 `.s` 和 `.d`，表示转换的来源为单精度/双精度浮点数。

- 指令格式：

  ```asm
  ftintrz.w.s $fd, $fj
  ftintrz.w.d $fd, $fj
  ftintrz.l.s $fd, $fj
  ftintrz.l.d $fd, $fj
  ```

- 行为：选择浮点寄存器 `$fj` 中的单精度/双精度浮点数转换为整数型/长整数型定点数，得到的整数型/长整数型定点数写入到**浮点寄存器** `$fd` 中

  ```c
  // ftintrz.w.s $fd, $fj
  FR[fd][31:0] = FP32convertToSint32(FR[fj][31:0], ROUND_ZERO)
  // ftintrz.w.d $fd, $fj
  FR[fd][31:0] = FP64convertToSint32(FR[fj], ROUND_ZERO)
  // ftintrz.l.s $fd, $fj
  FR[fd] = FP32convertToSint64(FR[fj][31:0], ROUND_ZERO)
  // ftintrz.l.d $fd, $fj
  FR[fd] = FP64convertToSint64(FR[fj], ROUND_ZERO)
  ```

### 浮点搬运指令

#### movgr2fr.w, movgr2fr.d

- `movgr2fr.w`

  - 指令格式：`movgr2fr.w $fd, $rj`

  - 行为：将通用寄存器 `$rj` 中的 `[31:0]` 位数据写入浮点寄存器 `$fd` 中的 `[31:0]` 位中，注意浮点寄存器 `$fd` 的 `[63:32]` 位值不确定

    ```c
    FR[fd][31:0] = GR[rj][31:0]
    ```

- `movgr2fr.d`

  - 指令格式：`movgr2fr.d $fd, $rj`

  - 行为：将通用寄存器 `$rj` 中的 `[63:0]` 位数据写入浮点寄存器 `$fd` 中

    ```c
    FR[fd] = GR[rj]
    ```

#### movfr2gr.s, movfr2gr.d

- `movfr2gr.s`

  - 指令格式：`movfr2gr.s $rd, $fj`

  - 行为：将浮点寄存器中的 `[31:0]` 位数据符号扩展后写入通用寄存器 `$rd`

    ```c
    GR[rd] = SignExtend(FR[fj][31:0], 64)
    ```

- `movgr2fr.s`

  - 指令格式：`movfr2gr.d $rd, $fj`

  - 行为：将浮点寄存器中的 `[63:0]` 位数据写入通用寄存器 `$rd`

    ```c
    GR[rd] = FR[fj]
    ```

### 浮点普通访存指令

在介绍浮点普通访存指令时，由于我们主要关注逻辑地址，因此会先不考虑逻辑地址和物理地址之间的转换，尽管这一转换在每条浮点普通访存指令的执行过程中都会发生。

#### fld.s, fld.d

- `fld.s`

  - 指令格式：`fld.s $fd, $rj, si12`

  - 访存地址：通用寄存器 `$rj` 中的值与符号扩展后的 12 比特立即数 `si12` 相加求和的结果

  - 行为：从内存取回一个字的数据写入浮点寄存器 `$fd` 的 `[31:0]` 位，注意 `$fd` 的 `[63:32]` 位值不确定

    ```c
    addr = GR[rj] + SignExtend(si12, 64) // 忽略了物理地址转换
    word = LoadMemory(addr, WORD)
    FR[fd] = word
    ```

- `fld.d`

  - 指令格式：`fld.d $fd, $rj, si12`

  - 访存地址：通用寄存器 `$rj` 中的值与符号扩展后的 12 比特立即数 `si12` 相加求和的结果

  - 行为：从内存取回一个双字的数据写入浮点寄存器 `$fd`

    ```c
    addr = GR[rj] + SignExtend(si12, 64) // 忽略了物理地址转换
    word = LoadMemory(addr, DOUBLEWORD)
    FR[fd] = doubleword
    ```

#### fst.s, fst.d

- `fst.s`

  - 指令格式：`fst.s $fd, $rj, si12`

  - 访存地址：通用寄存器 `$rj` 中的值与符号扩展后的 12 比特立即数 `si12` 相加求和的结果

  - 行为：将浮点寄存器 `$fd` 中的 `[31:0]` 位数据写入到内存中

    ```c
    addr = GR[rj] + SignExtend(si12, 64) // 忽略了物理地址转换
    StoreMemory(FR[fd][31:0], addr, WORD)
    ```

- `fst.d`

  - 指令格式：`fst.d $fd, $rj, si12`

  - 访存地址：通用寄存器 `$rj` 中的值与符号扩展后的 12 比特立即数 `si12` 相加求和的结果

  - 行为：将浮点寄存器 `$fd` 中的数据写入到内存中

    ```c
    addr = GR[rj] + SignExtend(si12, 64) // 忽略了物理地址转换
    StoreMemory(FR[fd][63:0], addr, DOUBLEWORD)
    ```

### 浮点比较指令

#### fcmp.cond.s, fcmp.cond.d

- 指令格式：`fcmp.cond.s $fccd, $fj, $fk` / `fcmp.cond.d $fccd, $fj, $fk`

- 行为：根据 `cond` 对 `$fj` 和 `$fk` 进行比较，将比较的结果存入[条件标志寄存器](#条件标志寄存器) `$fccd`

- `cond`：下面是一部分实验中可能会使用到的 `cond` 的信息，

  | 助记符 | `cond` |   含义   |         为真的条件         |
  | :----: | :----: | :------: | :------------------------: |
  | `seq`  | `0x5`  |   相等   |        `$fj == $fk`        |
  | `sne`  | `0x11` |   不等   | `$fj > $fk` 或 `$fj < $fk` |
  | `slt`  | `0x3`  |   小于   |        `$fj < $fk`         |
  | `sle`  | `0x7`  | 小于等于 |        `$fj <= $fk`        |

  如果你想了解全部的 22 种 `cond`，可以参考[《龙芯架构参考手册 - 卷一：基础架构》](https://github.com/loongson/LoongArch-Documentation/releases/latest/download/LoongArch-Vol1-v1.02-CN.pdf)的第 3.2.2 节

- 例子：`fcmp.slt.s $fcc0, $f0, $f1` 的行为如下

  ```python
  if float32(FR[0][31:0]) < float32(FR[1][31:0]):
      CFR[fcc0] = 1
  else：
      CFR[fcc0] = 0
  ```

### 浮点分支指令

!!! info "指令对齐"

    在龙芯架构中，所有指令均采用 32 位固定长度，且指令的地址都要求 4 字节边界对齐。当指令地址不对齐时将触发地址错例外。

!!! info "地址偏移量"

    对于 `bceqz` 和 `bcnez` 指令，如果在汇编代码中采用直接填入偏移值的方式（而不是填入 `.label`），则汇编表示中的立即数应该填入以字节为单位的偏移值，即指令格式中的 `offs21 << 2`。
    比如，对于下面的汇编指令
    ```asm
    .label_1:
        bceqz $fcc0, .label_2
    .label_2:
        add $zero, $zero, $zero
    ```
    它等价于
    ```asm
    .label_1:
        bceqz $fcc0, 0x4
    .label_2:
        add $zero, $zero, $zero
    ```

#### bceqz, bcnez

- 指令格式：`bceqz $fccj, offs21` / `bceqz $fccj, offs21`

- 跳转地址：将指令码中的 21 比特立即数 `offs21` 逻辑左移 2 位后再符号扩展，所得偏移值加上该分支指令的 `PC` 的和

- 行为：对[条件标志寄存器](#条件标志寄存器) `$fccj` 的值进行判断，如果等于/不等于 0 则跳转到目标地址，否则不跳转

  ```c
  // bceqz $fccj, offs21
  if CFR[fccj] == 0:
      PC = PC + SignExtend({ offs21, 2'b0 }, 64)
  // bceqz $fccj, offs21
  if CFR[fccj] != 0:
      PC = PC + SignExtend({ offs21, 2'b0 }, 64)
  ```

## GNU 汇编伪指令

由于我们生成的汇编代码最终会被 GNU 工具链中的 Assembler 翻译为机器语言，所以汇编代码中会包含一些 **GNU 汇编伪指令**。

<!-- TODO: Assembler Directives -->

如果想要更进一步了解 GNU 汇编伪指令，可以参考 [Assembler Directives](https://sourceware.org/binutils/docs-2.37/as/Pseudo-Ops.html#Pseudo-Ops)。

## Tips

- 如果想要将标签 `.label` 的地址写入通用寄存器，可以使用 `la.local` 伪指令，格式为 `la.local $fd, .label`
- 在跳转类指令中（`jirl` 除外），可以用目标标签 `.label` 代替立即数，在生成机器码的过程中汇编器会自动将其转换为对应的立即数
