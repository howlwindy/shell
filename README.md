[toc]

> [Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)

# Introduction

## What is Bash

- 是 Bourne-Again SHell 的缩写
- 很大程度上与 sh 兼容，并结合 ksh/csh 功能，符合 IEEE POSIX 标准的实现，总之比 sh 更好

## What is shell

- 执行命令的宏处理器(展开文本和符号以创建更大表达式的功能)
- UNIX 的 shell 既是解释器也是编程语言
  - 作为命令解释器，提供丰富的 GNU 使用程序集的用户界面
  - 作为编程语言，允许组合实用程序，创建包含命令的文件并成为命令本身
- 可交互可不交互
  - 交互模式：接收键盘输入
  - 非交互模式：执行从文件读取的命令
- 允许同步和异步执行 GNU 命令，重定向构造允许对命令的 I/O 进行细粒度的控制
  - 同步：等待上一个命令的执行完成
  - 异步：命令并行执行
- 提供不可能/不便实现的 build-in 命令
- 专门提供了交互式使用特性，而不是为了增强语言

# Definitions

> 通篇关键词

- POSIX
  > 基于 UNIX 的开放系统标准系列，Bash 主要关注标准 Shell 和 Utilities
- blank
  > space/tab
- builtin
  > shell 本身内部实现的命令
- control operator
  > 执行控制功能的 token：`newline` `||` `&&` `&` `;` `;;` `;&` `|` `|&` `(` `)`
- exit status
  > 命令返回给调用者的值，1 个字节，最大 255
- field
  > 文本单元，是 shell 展开的结果，展开后执行时，生成的字段被用作命令名和参数
- filename
  > 标识文件的字符串
- job
  > 由管道组成的一组流程及其派生的所有流程，都在同一流程组中
- job control
  > 用户可以有选择地停止(暂停)和重启(恢复)进程执行的机制
- matcharacter
  > 不加引号的字符，用于分隔单词：`space` `tab` `newline` `|` `&` `(` `)` `<` `>`
- name
  > 由字母/数字/下划线组成的标识符
- operator
  > 一种控制操作符或重定向操作符，至少包含一个不带引号的 metacharacter
- process group
  > 相关进程的集合，每个进程具有相同的进程组 ID
- process group ID
  > 在进程组的生存期内表示该进程组的唯一标识符
- reserved word
  > 保留字
- return status
  > exit status 的代名词
- signal
  > 内核可以通知进程在系统中发生事件的机制
- special buildin
  > 被 POSIX 归为特殊 shell 内置命令
- token
  > 被 shell 认为是单个单元的一系列字符，是一个单词或运算符
- word
  > 由 shell 作为一个单元处理的字符序列，单词不能包含没有引号的元字符

# Basic Shell Features

## Shell Syntax

> 输入对于 shell 的意义

1. shell 读取输入并分为单词和操作符
2. 使用引号规则来选择含义来分配单词和字符
3. 将标记解析为命令和其他结构，删除某些单词或字符的特殊含义，扩展其他字符，根据需要重定向 I/O，执行指定的命令，等待命令的退出状态并使退出状态可供进一步检查或处理

### Shell Operation

> shell 的基础操作

shell 读取命令并执行时的操作大致如下：

1. 从.sh 文件 | 作为参数提供给-c 调用选项的字符串 | terminal 输入
2. 按照引用中描述的引用规则，将输入分成单词和操作符，标记由 metacharacter 分隔
3. 将标记解析为简单命令 | 复杂命令
4. 执行各种 shell 扩展，将扩展分解为文件名列表 | 命令 | 参数
5. 执行必要的重定向并从实参列表中删除重定向操作符及其操作数
6. 执行命令
7. 可选的等待命令完成并收集其退出状态

### [Quoting](bash-reference-manual/3-1-2-quoting.sh)

> 去除字符的特殊含义

引号删除某些字符或单词对 shell 的特殊含义，可以禁用对特殊字符的特殊处理，防止保留字被识别，防止参数扩展

三种引用机制：

1. escape character `\`
2. single quotes `''`
3. double quotes `""`

#### Escape Character

> 去掉单个字符的特殊含义

转义其后的字符，消除特殊意义，\newline 除外

#### Single Quotes

> 禁止所有字符序列的解释

消除所有字符的特殊意义

#### Double Quotes

> 禁止大多数字符序列的解释

消除除$ ` \的特殊意义

#### ANSI-C Quoting

> 在引号字符串中扩展 ANSI-C

`$'string'`是特殊处理的字符串，会替换为 ANSI C 标椎的反斜杠转义字符

- `\a`
  > alert(bell)
- `\b`
  > backspace
- `\e` `\E`
  > escape
- `\f`
  > form feed
- `\n`
  > newline
- `\r`
  > return
- `\t`
  > 水平 tab
- `\v`
  > 垂直 tab
- `\\`
  > `\`
- `\'`
  > `'`
- `\"`
  > `"`
- `\?`
  > `?`
- `\nnn`
  > 八位 octal
- `\xHH`
  > 八位 hex
- `\uHHHH`
  > unicode
- `\UHHHHHHHH`
  > unicode
- `\cx`
  > control-x

#### Locale Translation

> 将字符串转化为不同语言

带有`$"string"`将该字符串根据当前地区进行翻译

### Comments

> 注释

没有启用交互式注释选项的交互式 shell 不允许注释

## Shell Commands

> 可用的命令类型

简单命令跟参数，由空格分隔
复杂命令由各种方式排列在一起的简单命令组成

### Reserved Words

> shell 中特殊意义的单词

用于开始和结束复合命令
没有被引号且是命令的第一个单词
如果是一个case或select的第三个单词，则会将in识别为保留字
如果in和do是for的第三个单词，则被认为是保留字

1. `if`
2. `then`
3. `elif`
4. `else`
5. `fi`
6. `time`
7. `for`
8. `in`
9. `until`
10. `while`
11. `do`
12. `done`
13. `case`
14. `esac`
15. `coproc`
16. `select`
17. `function`
18. `{}`
19. `}`
20. `[[`
21. `]]`
22. `!`

### Simple Commands

> 最常见的命令类型

### Pipelines

> 连接多个命令的输入和输出

### Lists

> 顺序的执行命令

### Compound Commands

> 控制流的命令

### Coprocesses

> 命令间的双向通信

### GNU Parallel

> 并行运行命令

## Shell Functions

> 命令以名字分组

## Shell Parameters

> shell 存储值的方式

## Shell Expansions

> 如何展开参数和各种可用扩展

## Redirections

> 控制 I/O 位置的方法

## Executing Commands

> 命令执行后的结果

## Shell Scripts

> 执行命令文件
