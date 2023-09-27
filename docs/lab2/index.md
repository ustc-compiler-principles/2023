# Lab2

本次实验需要同学们在 lab1 实现的 Cminusf 解析器基础上，完成从语法树向中间代码的自动化翻译过程。

## 文档

本次实验涉及若干文档，请根据实验进度和个人需要认真阅读。
必读：

- [LightIR 简介](../common/LightIR.md)
- [Cminusf 语义](../common/cminusf.md)
- [IR 自动化生成框架](./autogen.md)

选读：

- [C++ 简介](../common/simple_cpp.md)
- [logging 工具介绍](../common/logging.md)

## 实验内容
本次实验需要学生分阶段完成，并分阶段验收

### 阶段一

- 内容一：
  阅读[LightIR 预热](./warmup.md)并参考[LightIR 简介](../common/LightIR.md)，掌握手写 LightIR，使用 LightIR C++ 库生成 IR 的方法
- 内容二：
  阅读[访问者模式](./visitor_pattern.md)，理解 C++ 访问者模式的工作原理及遍历顺序

阶段一要求学生**回答[LightIR 预热](./warmup.md)与[访问者模式](./visitor_pattern.md)文档中的思考题**，回答内容保存为`answer.pdf`。

!!! warning "Deadline"

    **2023 年 10 月 12 日 23:59**


### 阶段二
  阅读[IR 自动化生成框架](./autogen.md)，补全 CminusfBuilder 类的所有 visit 函数，并通过`tests/2-ir-gen/autogen/testcases/`目录下 `lv0_1`, `lv0_2`, `lv1` 级别的测试样例

!!! warning "Deadline"
    
    **2023 年 10 月 19 日 23:59**

### 阶段三
  在阶段二的基础上，持续丰富 CminusfBuilder 类 visit 函数的实现，通过`tests/2-ir-gen/autogen/testcases/`目录下所有提供的测试样例

!!! warning "Deadline"
    
    **2023 年 10 月 26 日 23:59**

## 实验要求

请根据 Lab0 的内容，将[实验仓库](https://cscourse.ustc.edu.cn/vdir/Gitlab/compiler_staff/2023ustc-jianmu-compiler)设置为上游仓库，并获取本次实验更新的内容。

## 提交内容

- 阶段一：在希冀平台提交你的 `answer.pdf` 文件。
- 阶段二、三：在希冀平台提交你实验仓库的 url（如 `https://cscourse.ustc.edu.cn/vdir/Gitlab/xxx/2023ustc-jianmu-compiler.git`）。

