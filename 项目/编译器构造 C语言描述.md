# Micro语言
# 一、概述

Micro语言是一种简单的编程语言，以下是它的基本特性：

- 数据类型：
  - 唯一的数据类型是整型。
- 标识符：
  - 所有的标识符采用隐式声明，长度不超过32个字符。
  - 标识符由字母、数字和下划线组成。
- 文字常量：
  - 文字常量由一串数字组成。
- 注释：
  - 注释由"-"开始，并在当前行尾结束。
- 语句类型：
  - 赋值语句：
    - 格式：`ID := Expression;`
    - `Expression`是由标识符、文字常量以及`+`和`-`运算符组成，可以包含括号。
  - 输入/输出语句：
    - 输入：`read(List of IDs);`
    - 输出：`write(List of Expressions);`
  - 保留字包括：`begin`、`end`、`read`和`write`。
- 语句结束：
  - 每条语句以分号（`;`）结束。
- 程序体界定：
  - 程序体由`begin`和`end`界定。
- 源代码格式：
  - 源代码中每行结尾添加一个空格符；因此，词法记号不能跨行。







# 二、Micro编译器结构

---

本章的主题是一个基于图1-3描述的简单Micro编译器。该编译器采用单遍编译技术，其最重要的特征是不使用显式的中间表示。以下是编译器的各组件及其接口描述：
- **词法分析器**：从文本文件中读取源程序并产生记号表示流（第3章将严格定义）。实际上，并不需要在任意时刻都有实际的流存在，因为词法分析器实际上是一个由语法分析器调用的函数，每次调用都会产生一个记号表示。
- **语法分析器**：处理记号，直到遇到需要语义处理的语法结构。它随后直接调用语义例程。其中有些语义例程在其处理过程中使用记号表示。
- **语义例程**：为一个简单的三地址虚拟机产生汇编语言代码输出。因此，在Micro编译器结构中没有优化器，代码生成是通过从语义例程直接调用适当的支持例程来完成的。
- **符号表**：仅由语义例程使用。其接口在2.5.5节描述。







# 三、Micro词法分析器

---

```python
from typing import List
import argparse
import os
from enum import Enum

# 定义最大记号长度
MAX_TOKEN_LENGTH = 32
# 定义保留字列表
RESERVED_WORDS = ['begin', 'end', 'read', 'write']

# 定义记号类型枚举
class TokenType(Enum):
    BEGIN = 1
    END = 2
    READ = 3
    WRITE = 4
    ID = 5
    INTLITERAL = 6
    LPAREN = 7  # 左括号
    RPAREN = 8  # 右括号
    SEMICOLON = 9  # 分号
    COMMA = 10  # 逗号
    ASSIGNOP = 11  # 赋值运算符
    PLUSOP = 12  # 加号
    MINUSOP = 13  # 减号
    SCANEOF = 14  # 文件结束标记

# 输入流类，用于读取文件内容
class InputStream:
    def __init__(self, file_path: str):
        self.read_pointer = -1  # 初始化读取指针
        # 打开文件并读取内容
        with open(file_path, 'r') as file:
            self.file_content = file.read()

    # 获取当前位置的字符
    def get_char(self) -> str:
        self.read_pointer += 1
        # 如果读取指针在文件内容长度范围内，返回字符
        if self.read_pointer < len(self.file_content):
            return self.file_content[self.read_pointer]
        return ''  # 否则返回空字符串

    # 回退一个字符
    def unget_char(self) -> None:
        if self.read_pointer > -1:
            self.read_pointer -= 1

# 词法分析器函数
def scanner() -> TokenType:
    # 将字符添加到缓冲区
    def add_to_buffer(char: str) -> None:
        if len(token_buffer) == MAX_TOKEN_LENGTH:
            raise Exception('Token length exceeds MAX_TOKEN_LENGTH, which is 32.')
        token_buffer.append(char)

    # 清空缓冲区
    def clear_buffer() -> None:
        token_buffer.clear()

    # 检查缓冲区中的字符串是否为保留字
    def is_reserved_word() -> TokenType:
        token_str = ''.join(token_buffer)
        if token_str in RESERVED_WORDS:
            return TokenType[token_str.upper()]
        return TokenType.ID

    global input_stream
    char = ''
    next_char = ''

    clear_buffer()  # 清空缓冲区

    while True:
        char = input_stream.get_char()
        if not char:
            return TokenType.SCANEOF  # 如果没有字符了，返回文件结束标记

        if char.isspace():
            continue  # 跳过空白字符

        if char.isalpha():
            add_to_buffer(char)
            next_char = input_stream.get_char()
            while next_char.isalnum() or next_char == '_':
                add_to_buffer(next_char)
                next_char = input_stream.get_char()
            input_stream.unget_char()
            return is_reserved_word()  # 检查是否为保留字

        if char.isdigit():
            add_to_buffer(char)
            next_char = input_stream.get_char()
            while next_char.isdigit():
                add_to_buffer(next_char)
                next_char = input_stream.get_char()
            input_stream.unget_char()
            return TokenType.INTLITERAL  # 返回整型字面量

        # 根据字符返回相应的记号类型
        if char == '(':
            return TokenType.LPAREN
        if char == ')':
            return TokenType.RPAREN
        if char == ';':
            return TokenType.SEMICOLON
        if char == ',':
            return TokenType.COMMA
        if char == '+':
            return TokenType.PLUSOP
        if char == '-':
            return TokenType.MINUSOP
        if char == ':':
            next_char = input_stream.get_char()
            if not next_char:
                return TokenType.SCANEOF
            if next_char == '=':
                return TokenType.ASSIGNOP
            raise Exception('Lexical error occurs.')  # 如果不是赋值运算符，则抛出词法错误

        raise Exception('Lexical error occurs.')  # 如果字符不匹配任何记号，则抛出词法错误

# 主程序入口
if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('file_path', type=str, default='code', help='Absolute or relative code file path.')
    args = parser.parse_args()
    file_path = args.file_path
    input_stream = InputStream(file_path)

    token_buffer: List[str] = []  # 初始化记号缓冲区

    while True:
        # 调用 scanner 函数获取记号类型
        tokentype = scanner()
        # 打印记号类型
        print(tokentype)
        # 如果遇到文件结束标记，则退出循环
        if tokentype == TokenType.SCANEOF:
	        break
```







# 四、Micro语法分析器

---

EBNF -> Extended BNF，拓展巴克斯诺尔范式，不改变BNF的表达能力，面向人类简化语法规则的编写。

| 记号      | 意义    |
| ------- | ----- |
| =       | 定义    |
| ,       | 连接符   |
| ;       | 结束符   |
| \|      | 或     |
| [...]   | 可选    |
| {...}   | 重复    |
| (...)   | 分组    |
| "..."   | 终端字符串 |
| '...'   | 终端字符串 |
| (*...*) | 注释    |
| ?...?   | 特殊序列  |
| -       | 除外    |

Micro的语法有以下：

| 序号  | 产生式                                              |
| --- | ------------------------------------------------ |
| 1   | `<program> -> begin <statement list> end`        |
| 2   | `<statement list> -> <statement> <statement>`    |
| 3   | `<statement> -> ID : <expression> ;`             |
| 4   | `<statement> -> read ( <id list> ) ;`            |
| 5   | `<statement> -> write ( <expr list> ) ;`         |
| 6   | `<id list> -> ID {, ID}`                         |
| 7   | `<expr list> -> <expression> {, <expression>}`   |
| 8   | `<expression> -> <primary> {<add op> <primary>}` |
| 9   | `<primary> -> ( <expression> )`                  |
| 10  | `<primary> -> ID`                                |
| 11  | `<primary> -> INTLITERAL`                        |
| 12  | `<add op> -> PLUSOP`                             |
| 13  | `<add op> -> MINUSOP`                            |
| 14  | `<system goal> -> <program> SCANEOF`             |


> **怎么在CFG中实现运算符的优先级**（不过Micro没有*/不需要考虑），答案就是将表达式中的成分分类型：
> $$\begin{flalign}
> & <expression> → <factor>\{<add op><factor>\} \\
> & <factor> → <primary>\{<mult op><primary>\} \\
> & <primary> → (<expression>) \\
> & <primary> → ID \\
> & <primary> → INTLITERAL
> \end{flalign}$$
> 可以看到最高层expression被定义为一个factor或者一个addop和结果，这样的结果就是，addop将会被最后求值。而addop的参数是factor，factor被定义为primary或者primary的multop的结果。

