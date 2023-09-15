## Flex

## 简介

最初的词法分析器生成程序Lex由Mike Lesk和当时在AT&T实习的Eric Schmidt于1975年合作完成。Lex可单独使用或与Johnson的yacc协同工作。然而，Lex以低效率和bug而闻名。在20世纪80年代，Lawrence Berkeley实验室的Vern Paxson以C重新编写了Lex，命名为Flex（Fast Lexical Analyzer Generator）。Flex现在是SourceForge的一个项目，用于C/C++的词法扫描生成器。Flex支持使用正则表达式描述词法模式。



Flex执行过程如下：

![img](http://alumni.cs.ucr.edu/~lgao/teaching/Img/flex.jpg)

1. 首先，Flex源程序中的规则被转换成状态转换图，生成对应的代码，包括核心的yylex()函数，保存在lex.yy.c文件中。Flex源程序通常以.l为后缀，按照Flex语法编写，用于描述词法分析器。

2. 生成的lex.yy.c文件可以通过C编译为可执行文件。

3. 最终，可执行文件将输入流解析成一系列的标记（tokens）。



## Flex语言

Flex源程序的结构如下：

```c
声明部分

规则部分

C代码部分
```

- **声明部分**包含名称声明和选项设置，`%{`和`%}`之间的内容会被原样复制到生成的C文件头部，可用于编写C代码，如头文件声明和变量定义等。
- **规则部分**位于两个`%%`之间，包括多条规则，每个规则由正则表达式定义的模式和与之匹配的C代码动作组成。当词法分析程序识别出某模式时，执行相应的C代码。
- **C代码部分**可包括`main()`函数，用于调用`yylex()`执行词法分析。`yylex()`是由Flex生成的词法分析例程，默认从stdin读取输入文本。

下面我们结合具体的例子，来看Flex的源程序的三部分结构：

```c
/* %option noyywrap功能较为复杂，同学们自行了解 */
%option noyywrap
%{
/* Flex源程序采样类C的语法和规则 */
/* 以下是声明部分，`%{`和`%}`之间的内容会被原样赋值到生成的C文件头部
	包括该条注释内容 */
#include <string.h>
int chars = 0;
int words = 0;
%}

/* 以下是规则部分，在规则部分写注释不能顶格写 */
/* 每条规则由正则表达式和动作组成 */
/* 第一条规则匹配纯字母的字符串，并统计字母个数和字符串个数
                       其中yytext为匹配到的token */
/* 第二条规则匹配其他字符或字符串并执行空动作 */
%%
 /* 在规则部分，不要顶格写注释 */
[a-zA-Z]+  { chars += strlen(yytext); words++; }
.          {}
%%

/* 以下为C代码部分 */
int main()
{
    /* yylex() 是由Flex自行生成的，用于执行 */
    yylex();
    /* 对于stdin输入匹配结束，执行其他操作 */
    printf("look, I find %d words of %d chars\n", words, chars);
    return 0;
}
```



## 编译和执行

下面，我们将演示如何在Ubuntu系统上编译和运行Flex源程序，假设以上的Flex源程序被保存为`wc.l`。

首先，我们执行以下命令将Flex源程序转化为c程序：

```shell
$ flex wc.l
```

接着，我们查看生成的`lex.yy.c`文件：

```shell
$ ls lex.yy.c
lex.yy.c
```

`lex.yy.c`文件比较复杂，你可以简要浏览一下：

```shell
$ cat lex.yy.c
......
```

接下来，我们编译`lex.yy.c`生成可执行文件：

```shell
$ gcc lex.yy.c -o lexer
```

此时，将生成一个可执行文件`lexer`：

```shell
$ ls lexer
lexer
```

最后，执行可执行文件并输入`hello world!`：

```shell
$ ./lexer 
hello world!
^D # 如果使用stdin作为输入，按Ctrl+D退出
look, I find 2 words of 10 chars
```

恭喜，你已成功使用Flex创建了一个简单的分析器！



## 扩展阅读

Flex自带详细手册，通过以下终端命令打开：

```shell
$ info flex
# 按q键退出
```

特别推荐仔细阅读章节`Pattern`和`Matching`。



## 思考题

1. 如果同时以下规则和动作，对于字符串`+=`，哪条规则会被出发，并尝试解释理由。

   ``` c
   %%
   "\+" { return ADD; }
   "=" { return ASSIGN; }
   "\+=" { return ASSIGNADD; }
   %%
   ```

2. 如果同时以下规则和动作，对于字符串`ABC`，哪条规则会被出发，并尝试解释理由。

   ```c
   %%
   "[a-zA-Z]+" {return WORDS; }    
   "ABC" { return ABC; }
   %%
   ```

提示：你可以编写相关程序，实际进行执行。具体原因在Flex自带手册的`Matching`章节中有详细说明。

