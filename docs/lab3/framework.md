# 后端框架介绍

本次更新的相关文件如下：

```
.
├── ...
├── include
│   ├── ...
│   └── codegen
│       ├── ASMInstruction.hpp	# 描述汇编指令
│       ├── CodeGen.hpp			# 后端框架顶层设计
│       ├── CodeGenUtil.hpp		# 一些辅助函数及宏的定义
│       └── Register.hpp		# 描述寄存器
├── src
│   ├── ...
│   └── codegen
│       ├── CMakeLists.txt
│       ├── CodeGen.cpp
│       └── Register.cpp
└── tests
    ├── ...
    └── 3-codegen/*				# lab3 测试
```

## 极简框架

本次实验的框架非常简单，你只需要少量的代码阅读就能快速进入代码撰写的环节。

顶层的 `Codegen` 类只维护了如下成员：

```cpp
class CodeGen {
    // ...
  private:
    struct { ... } context;				// 类似 lab2 的 context
    Module *m;							// IR 模块
    std::list<ASMInstruction> output;	// 汇编指令列表
};
```

几个上层函数如下：

```cpp
class CodeGen {
    // ...
  public:
    explicit CodeGen(Module *module) : m(module) {}	// 构造函数
    std::string print() const;						// 输出汇编指令
    void run();										// 代码生成
}
```

你需要了解或者实现的是下面一系列函数：

```cpp
class CodeGen {
    // ...
  private:
    // 变量分配
    void allocate();

	/* 助教准备的辅助函数 */
    // 将数据在寄存器和栈帧间搬移。下边的章节将详细介绍
	void load_xxx(...);
    void store_xxx(...);
    // 添加汇编指令
    void append_inst(...);
    // 根据基本块名字生成汇编指令段的标号
	static std::string label_name(BasicBlock *bb);

	// 需要补全的部分，进行代码生成的各个环节
    void gen_xxx(...);
};
```

初始代码已经为你处理好了一些繁琐的细节，如全局变量的定义及初始化、汇编中 `section` `type` 的定义等，所以你可以把重点放在栈式分配的实现中。

## 基本类描述

### 指令类

在上述代码中，你已经看到了指令类 `ASMInstruction` 的用处：在 `CodeGen` 中以 `std::list` 形式组织。指令类的代码定义如下：

```cpp
struct ASMInstruction {
    enum InstType { Instruction, Atrribute, Label, Comment } type;
    std::string content;

    explicit ASMInstruction(std::string s, InstType ty = Instruction);
    std::string format() const;
};
```

一个指令类实例，对应汇编代码的一行，在 `content` 中保存其核心内容，通过枚举类型 `type` 记录其用途。

在指令输出时，会调用 `format()` 函数，根据类型 `type` 添加缩进或其他字符，详细请查阅 `ASMInstruction::format()` 的实现。

举个例子，`ASMInstruction("some debug info", ASMInstruction::Comment)` 定义了一个指令类实例，其用途是注释， `format()` 的返回结果是如下字符串：`"#some debug info\n"`。

### 寄存器类

对寄存器的抽象，聚焦在区分不同物理寄存器上。

寄存器分为通用寄存器 `Reg` 和浮点寄存器 `FReg`，以下是 `FReg` 的代码定义：

```cpp
struct FReg {
    unsigned id; // 0 <= id <= 31

    explicit FReg(unsigned i);
    bool operator==(const FReg &other);

    std::string print() const;

    static FReg fa(unsigned i);
    static FReg ft(unsigned i);
    static FReg fs(unsigned i);
};
```

`Reg` 的定义与之基本一致，两个类都会在内部保存 `id`，用来比较寄存器是否相同和打印输出。

举例：

- `FReg(0)` 定义了寄存器 `$f0` 的实例，其打印的结果是 `"$fa0"`
- 为了获得 `$ft0` 的实例，你可以使用 `FReg(8)`，也可以使用更方便的`FReg::ft(0)`

## 辅助函数

助教提供了很多辅助函数，来帮助你更舒适地完成本次实验。

### load/store

阅读过[栈式分配](./guidance.md/#栈式分配)的介绍后，一些同学会敏锐地意识到，我们生成的汇编中，内存交互会非常频繁。

针对此，我们提供了如下函数，用于方便地提取数据至寄存器和将寄存器数据保存至栈上。

```cpp
class CodeGen {
	//...
  private:
	// 向寄存器中装载数据
    void load_to_greg(Value *, const Reg &);
    void load_to_freg(Value *, const FReg &);
    void load_from_stack_to_greg(Value *, const Reg &);
    // 将寄存器中的数据保存回栈上
    void store_from_greg(Value *, const Reg &);
    void store_from_freg(Value *, const FReg &);
};
```

你只需要提供 IR 中的 Value 指针，同时指定目标寄存器，即可方便地完成数据的 load/store.

事实上，寄存器另一大数据来源，还有立即数的提取。我们也提供了一些辅助函数：

```cpp
class CodeGen{
    // ...
  private:
	// 向寄存器中加载立即数
    void load_int32_imm(int32_t, const Reg &);
    void load_large_int32(int32_t, const Reg &);
    void load_large_int64(int64_t, const Reg &);
    void load_float_imm(float, const FReg &);
};
```

关于这些 API，虽然在这里给出了一些注释，它们看起来很好用，但是仍然不建议你开箱即用，希望你能简单阅读一下源码，理解其中在做什么后，再开始你的实现。

### append_inst()

`CodeGen` 中以 `std::list` 的形式组织 `ASMInstruction`，`append_inst()` 接口就是用来添加新的 `ASMInstruction`。

这里有两个版本的`append_inst()`：

- 你可以直接按照 `ASMInstruction` 的构造函数添加指令，如：

  ```cpp
  // 第二个参数的默认值即为 ASMInstruction::Instruction
  // append_inst("st.d $ra, $sp, -8", ASMInstruction::Instruction);
  append_inst("st.d $ra, $sp, -8");
  ```

- 也可以使用二次封装后的版本：

  ```cpp
  // 最后一个参数的默认值即为 ASMInstruction::Instruction
  // append_inst("st.d", {"$ra", "$sp", "-8"}, ASMInstruction::Instruction);
  append_inst("st.d", {"$ra", "$sp", "-8"});
  ```

  这么看还没有什么区别，在 `include/codegen/CodeGenUtil.hpp` 中，我们定义了一系列宏定义，你可以去看看，使用这些宏定义，结合其他 API，上述调用会变得不太一样：

  ```cpp
  append_inst(STORE DOUBLE, {Reg::ra().print(), Reg::sp().print(), "-8"});
  ```

## 变量分配

关于变量分配，我们需要为函数中每个变量分配一段栈空间，这个并没有固定的标准，框架中提供了一个已经经过验证的版本，即 `CodeGen::allocate()`。你可以设计自己喜欢的 `allocate()`，但是这不被推荐，因为我们不能保证框架的其他部分（主要是辅助函数）和你的设计兼容。

初始代码中的 `allocate()` 是这样分配空间的：

- 记录每个变量相对于栈底的偏移，由于栈从高向低生长，所以这个偏移量为负数
- 固定备份两个寄存器：`$ra` 和 `$fp`
- 备份函数参数
- 为每个存在定值的指令分配相应的空间
- 对于 `alloca` 指令，`alloca` 本身的定值为指针类型，`alloca` 的空间紧挨着这个这个指针，在更靠近栈顶的位置

即我们的栈帧结构如下：

![](figs/stack-frame.png)
