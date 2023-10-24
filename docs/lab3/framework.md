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
│       ├── CodeGen.cpp		<-- 你需要修改的文件
│       └── Register.cpp
└── tests
    ├── ...
    └── 3-codegen/*				# lab3 测试
```

## 极简框架

本次实验的框架相对简单，你只需要少量的代码阅读就能快速进入代码撰写的环节。

顶层的 `Codegen` 类只维护了如下成员：

```cpp
class CodeGen {
    // ...
  private:
    struct { ... } context;				// 类似 lab2 的 context，用于保存翻译过程中的上下文信息，如当前所在函数
    Module *m;							// 输入的 IR 模块
    std::list<ASMInstruction> output;	// 生成的汇编指令
};
```

几个上层函数如下：

```cpp
class CodeGen {
    // ...
  public:
    explicit CodeGen(Module *module) : m(module) {}	// 构造函数
    std::string print() const;						// 将汇编指令格式化输出
    void run();										// 后端代码生成的入口函数
}
```

你需要了解或者实现的是下面一系列函数：

```cpp
class CodeGen {
    // ...
  private:
    // 栈式分配的变量分配环节，将在函数翻译开始时调用
    void allocate();

	/*=== 以下为助教准备的辅助函数 ===*/
    // 将数据在寄存器和栈帧间搬移。下边的章节将详细介绍
	void load_xxx(...);
    void store_xxx(...);
    // 添加汇编指令
    void append_inst(...);
    // 基本块在汇编程序中的名字
	static std::string label_name(BasicBlock *bb);
	/*=== 以上为助教准备的辅助函数 ===*/

	// 需要补全的部分，进行代码生成的各个环节
    void gen_xxx(...);
};
```

初始代码已经为你处理好了一些繁琐的细节，如全局变量的定义及初始化、汇编中 `section` `type` 的定义等，所以你可以把重点放在栈式分配的实现中。

## 基本类描述

框架对后端中的指令和寄存器进行了抽象，下面将依次介绍这两个基本类。

### 指令类

指令类 `ASMInstruction` 是用来描述一行汇编指令的，在 `CodeGen` 中以 `std::list` 形式组织。指令类的代码定义如下：

```cpp
struct ASMInstruction {
    enum InstType {
        Instruction,	// 汇编指令
        Atrribute,		// 汇编伪指令、描述符等非常规指令
        Label,			// 汇编中的 label
        Comment			// 注释
    } type; 			// 用来描述指令的用途，会被下面的 format 函数使用

    std::string content; // 汇编代码，不包含换行符等格式化的信息

    explicit ASMInstruction(std::string s, InstType ty = Instruction); // 构造函数
    std::string format() const; // 根据 type 对 content 进行格式化（如添加缩进、换行符等）
};
```

举个例子，`ASMInstruction("some debug info", ASMInstruction::Comment)` 定义了一个指令类实例，其用途是注释， `format()` 的返回结果是如下字符串：`"#some debug info\n"`。

### 寄存器类

寄存器分为通用寄存器 `Reg` 、浮点寄存器 `FReg` 和条件标志寄存器 `CFReg`。

以下是 `FReg` 的代码定义，`Reg` 与 `CFReg` 的定义与之类似：

```cpp
struct FReg {
    unsigned id; // 0 <= id <= 31

    explicit FReg(unsigned i);
    bool operator==(const FReg &other);

    std::string print() const;	// 根据 id 返回寄存器别名，如 "$fa0" 而不是 "$f0"

    static FReg fa(unsigned i);	// 得到寄存器 $faN
    static FReg ft(unsigned i);	// 得到寄存器 $ftN
    static FReg fs(unsigned i);	// 得到寄存器 $fsN
};
```

举例：

- `FReg(0)` 定义了寄存器 `$f0` 的实例，`print()` 的结果是 `"$fa0"`
- 为了获得 `$ft0` 的实例，你可以使用 `FReg(8)`，也可以使用更方便的`FReg::ft(0)`

## 辅助函数

在顶层的 `CodeGen` 类中，助教提供了很多辅助函数，主要是进行数据装载/写回的 load/store 相关和追加指令相关两大类，来帮助你更舒适地完成本次实验。

### load/store

阅读过[栈式分配介绍](stack_allocation.md)后，你将认同：我们生成的汇编中，load、store 的使用会非常频繁。

针对此，我们提供了如下函数，用于方便地提取数据至寄存器和将寄存器数据保存至栈上。

```cpp
class CodeGen {
	//...
  private:
	// 向寄存器中装载数据
    void load_to_greg(Value *, const Reg &);	// 将 IR 中的 Value 加载到整形寄存器中
    void load_to_freg(Value *, const FReg &);	// 将 IR 中的 Value 加载到浮点寄存器中

    // 将寄存器中的数据保存回栈上
    void store_from_greg(Value *, const Reg &);	// 将整形寄存器中的数据保存至 IR 中 Value 对应的栈帧位置
    void store_from_freg(Value *, const FReg &);// 将浮点寄存器中的数据保存至 IR 中 Value 对应的栈帧位置
};
```

你只需要提供 IR 中的 `Value` 指针，同时指定目标寄存器，即可方便地完成数据的加载或备份。

事实上，寄存器的另一大数据来源，还有立即数的提取。对于 12bit 能够表示的整形立即数，你可以直接使用 `$dest = $zero + imm` 的形式，对于比较复杂的大立即数提取及浮点立即数提取，我们提供了一些辅助函数：

```cpp
class CodeGen{
    // ...
  private:
	// 向寄存器中加载立即数
    void load_large_int32(int32_t, const Reg &);	// 提取 32 bit 的整数
    void load_large_int64(int64_t, const Reg &);	// 提取 64 bit 的整数
    void load_float_imm(float, const FReg &);		// 提取单精度浮点数（32bit）
};
```

关于这些 API，虽然在这里给出了一些注释，它们看起来很好用，但是仍然不建议你开箱即用，希望你能简单阅读一下源码，理解其中在做什么后，再拿来使用。

### append_inst()

`CodeGen` 中以 `std::list` 的形式组织 `ASMInstruction`，`append_inst()` 接口就是用来添加新的 `ASMInstruction`。

这里有两个版本的`append_inst()`：

- 你可以直接按照 `ASMInstruction` 的构造函数添加指令，如：

  ```cpp
  append_inst("st.d $ra, $sp, -8", ASMInstruction::Instruction);
  // 第二个参数的默认值即为 ASMInstruction::Instruction，所以下边的代码等价
  append_inst("st.d $ra, $sp, -8");
  ```

- 也可以使用二次封装后的版本：

  ```cpp
  append_inst("st.d", {"$ra", "$sp", "-8"}, ASMInstruction::Instruction);
  // 最后一个参数的默认值即为 ASMInstruction::Instruction，所以下边的代码等价
  append_inst("st.d", {"$ra", "$sp", "-8"});
  ```

  这么看还没有什么区别，在 `include/codegen/CodeGenUtil.hpp` 中，我们定义了一系列宏定义，你可以去看看，使用这些宏定义，结合其他 API，上述调用会变得不太一样：

  ```cpp
  append_inst(STORE DOUBLE, {Reg::ra().print(), Reg::sp().print(), "-8"});
  ```

## 变量分配

关于变量分配的实验方案，我们已经在[栈式分配介绍](stack_allocation.md/#实验方案)中有过提及，初始代码已经为你实现，在 `src/codegen/CodeGen.cpp` 中的 `CodeGen::allocate()`。

你可以设计自己喜欢的 `allocate()`，但是这不被推荐，因为我们不能保证框架的其他部分（主要是辅助函数）和你的设计兼容。
