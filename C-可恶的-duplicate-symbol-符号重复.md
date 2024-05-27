---
title: C++ 可恶的 duplicate symbol (符号重复)
date: 2024-04-01 09:23:30
tags:
- Cpp
- 编译原理
thumbnail: "https://darkflamemasterdev.github.io/img/language_headimg.png"
---

## 背景

我之前是主写 Java Kotlin 的，C++ 写的少之甚少

之前写 OpenGL 的时候，就一直被 duplicate symbol 这个问题困扰

现在我终于找到了解决方法

## 原理

首先，C++ 的编译器本身也是一段程序，而且不同编译器，编译成功与否可能会有细微的差别，但几乎都是相同的

1. 如果你在两个以上文件声明了同样的函数（方法），就会很容易产生重复。
2. 预处理阶段（Preprocessing），预处理指令会对源码文件（cpp 文件）进行临时扩充，预处理命令以 `#` 开头，例如：`#include`，`#define`，`#ifdef`等
   对于 `#define` 指令，编译器将源码中的宏替换成宏定义中的内容；对于 `#if`、`#ifdef` 和 `#ifndef` 指令，编译器将有选择地跳过或选中部分源代码；而对于 `#include` 指令，编译器将把对应的库的源码插入到当前源代码中——这通常是一些通用的声明。被 `#include` 指令引入的头文件（ .h ）往往会包含大量的代码，你引入的越多，最后生成的预编译文件就越大。总的来说，预编译过的文件会比原来的 C++ 源码更大一些。
3. 编译编译和汇编阶段（Compilation & assembly），预处理结束后，将预处理好的文件编译成 object 目标文，也就是 .o 文件。
4. 链接阶段（linking），将 .o 文件里 未定义标识符的引用全部替换成它们对应的正确地址，地址对应上，这就算是链接上了，这一步就是我们代码编译过程中产生 duplicate symbol（重复）的地方。

也就是说，在编译器进行链接（linking）的时候，由于某个方法我们声明了多次，所以它并不知道要链接到那个地址，所以就报错了

## 解决办法

### 消除 symbol，直接不声明了

1. 将 `method` 声明为 `inline`。`inline`方法会被直接 copy 到方法调用处，这个 Kotlin 也有类似的语法
2. 宏定义

   ```C++
   #define LOG(s) (std::cout << s << std::endl)
   ```

   这个也是类似，文本替换

### 声明为静态

静态函数会只在当前 object目标文件（.o文件）可见
