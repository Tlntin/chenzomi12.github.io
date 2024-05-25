<!--Copyright © ZOMI 适用于[License](https://github.com/chenzomi12/AISystem)版权许可-->

# LLVM 架构设计和原理

在上一节中，我们详细探讨了 GCC 的编译过程和原理。然而，由于 GCC 存在代码耦合度高、难以进行独立操作以及庞大的代码量等缺点。正是由于对这些问题的意识，人们开始期待新一代编译器的出现。在本节，我们将深入研究 LLVM 的架构设计和原理，以探索其与 GCC 不同之处。

## LLVM 发展历程

在早期的 Apple MAC 电脑选择使用 GNU 编译器集合（GCC）作为官方编译器。尽管 GCC 在开源世界中表现良好，但苹果对编译工具提出了更高要求。苹果在增强 Objective-C 和 C 语言方面投入了大量努力，但 GCC 开发者对苹果在 Objective-C 支持方面的努力表示不满。因此，苹果基于 GCC 分为两个分支进行独立开发，导致苹果的编译器版本明显落后于官方 GCC 版本。

![](./../images/03Compiler01Tradition/04LLVM01.png)

另一方面，GCC 的代码过于耦合，难以独立开发。随着版本更新，代码质量逐渐下降，而 GCC 无法以模块化方式调用实现 Apple 渴望的更好集成开发环境支持，这限制了 Apple 在编译器领域的发展。

面对这些问题，苹果一直寻找高效、模块化和更自由的开源替代方案。最终，苹果聘请了编译器领域的专家克里斯·拉特纳来领导 LLVM 项目的实现，标志着 LLVM 编译器的诞生。

![Chris Lattner](./../images/03Compiler01Tradition/04LLVM02.png)

LLVM 项目起源于 2000 年伊利诺伊大学厄巴纳-香槟分校的维克拉姆·艾夫（Vikram Adve）和克里斯·拉特纳（Chris Lattner）的研究，旨在为所有静态和动态语言创建动态编译技术。LLVM 是以 BSD 许可证开发的开源软件。2005 年，苹果公司雇用了克里斯·拉特纳及其团队为 macOS 和 iOS 开发工具，LLVM 成为了这些平台开发工具的一部分。

项目最初的命名来源于底层虚拟机（Low Level Virtual Machine）的首字母缩写。随着 LLVM 项目的发展，该缩写引起了混淆，因为项目范围不仅限于创建虚拟机。随着 LLVM 发展壮大，它成为了许多编译工具和低级工具技术的统称，这使得缩写变得不再合适。于是开发者决定摒弃缩写的含义，现在 LLVM 已经成为一个品牌，用于指代 LLVM 项目下的所有子项目，包括 LLVM 中介码（LLVM IR）、LLVM 调试工具、LLVM C++标准库等。

从 9.0.0 版本开始，LLVM 改用带有 LLVM 额外条款的 Apache 许可证 2.0。自 2019 年 10 月起，LLVM 项目的代码存储已正式迁移到 GitHub[<sup>3</sup>](#ref3)。

LLVM 项目已经迅速发展成为一个庞大的编译器工具集合。LLVM 激发了许多人为多种编程语言开发新的编译器，其中最引人注目的之一是 Clang。作为一个新的编译器，Clang 提供对 C、Objective-C 和 C++ 的支持，并且得到了苹果公司的大力支持。Clang 的目标是取代 GCC 在系统中的 C 和 Objective-C 编译器，它能够更轻松地集成到现代开发环境（IDE）中，并且支持线程的更好处理。从 Clang 3.8 版本开始，它还开始支持 OpenMP。GCC 在 Objective-C 方面的发展已经停滞，苹果已经将其支持转移到其他维护分支上。

在 2021 年 jetbrains 开发者调查中表示 GCC 编译器拥有 78%的用户使用率，Clean 编译器有 43% 的用户使用率。

![](../images/03Compiler01Tradition/04LLVM05.png)

> 由于 LLVM 对产业的重大贡献，计算机协会在 2012 年授予维克拉姆·艾夫、克里斯·拉特纳和 Evan ChengACM 软件系统奖。

====== 我稍微做了结构的调整，这里可以再 review 以下，整体在调整一下细节使得更加通顺。

## LLVM 架构特点

======= LLVM 的架构特点是？一段话高度总结一下哈。

### LLVM 组件独立

LLVM 具有一个重要的特点，就是它的组件独立性和库化架构。使用 LLVM，前端工程师在需要添加一门新的编程语言时，只需实现相应的前端，而不需要修改后端部分。这是因为后端只需将中间表示（IR）翻译成目标平台的机器码即可。

对于后端工程师来说，他们只需将目标硬件的特性如寄存器、硬件调度以及指令调度对接到 IR 上，而无需修改前端部分。这种灵活的架构使得编译器的前端和后端工程师可以相对独立地开展工作，大大提高了开发效率和维护性。

在 LLVM 中，IR 扮演着至关重要的角色。它是一种类似汇编语言的底层语言，但具有强类型和精简指令集的特点（RISC），并对目标指令集进行了抽象。例如，在 IR 中，目标指令集的函数调用惯例会被抽象为 call 和 ret 指令，并使用明确的参数。

此外，LLVM 支持三种不同的 IR 表达形式：人类可读的汇编形式、在 C++中的对象形式以及序列化后的 bitcode 形式。这种多样化的表达形式使得开发人员更好地理解和处理 IR，进而实现更加灵活和高效的编译工作。通过 IR 的抽象和统一，LLVM 极大地创新了编译体系的概念，为编程语言的快速开发和跨平台支持提供了强大的基础。

### LLVM 中间表达

LLVM 提供了一套适合编译器系统的中间语言（Intermediate Representation，IR），围绕这个中间语言进行了大量的变换和优化。经过这些变换和优化，IR 可以被转换为目标平台相关的汇编语言代码。  

与传统 GCC 的前端直接对应于后端不同，LLVM 的 IR 是统一的，可以适用于多种平台、进行优化和代码生成。

> 根据 2011 年的测试结果，LLVM 的性能在运行时平均比 GCC 低 10%。2013 年的测试显示，LLVM 能够编译出与 GCC 性能相近的执行代码。

GCC：

![GCC](./../images/03Compiler01Tradition/04LLVM03.png) 

LLVM：

![LLVM 的 IR](./../images/03Compiler01Tradition/04LLVM04.png)

LLVM IR 的优点包括： 

1. 更独立：LLVM IR 设计为可以在编译器之外的任意工具中重用，使得可以轻松集成其他类型的工具，例如静态分析器和插桩器。 

2. 更正式：拥有明确定义和规范化的 C++ API，这使得处理、转换和分析变得更加容易。 

3. 更硬件：LLVM IR 提供了类似 RISCV 的模拟指令集和强类型系统，实现了其“通用表示”的目的。足够底层的指令和细粒度的类型使得上层语言和 IR 的隔离变得容易，同时 IR 的行为更加贴近硬件，使得在 LLVM IR 上的进一步分析成为可能。

======= 很多内容看上去像是翻译的，可以自己通读一下文章，然后改改。

## LLVM 整体架构

LLVM 整体架构，涵盖从指令选择到最终汇编指令生成的完整优化流程。实际在使用过程中使用 C/C++/Obj-C 都会用到 Clang 的前端，往下使用的是 LLVM 的优化器后端。

![编译器](../images/03Compiler01Tradition/04LLVM06.png)

所以用户看到的就只是 Clang 的前端，而后面 LLVM 的优化器和后端都隐藏在幕后。

![编译器](../images/03Compiler01Tradition/04LLVM07.png)

首先我们以 C/C++/Obj-C 高级语言作为输入，输入到 Clang 前端，Clang 前端会对这些高级语言做一些词法分析、语法分析、语义分析等操作，生成抽象语法树（AST）。然后 Clang 会将 AST 转换为 LLVM 的中间表示（IR）。

LLVM 的中间表示（IR）连接前端和后端。然后 LLVM 的优化器会对 IR 进行优化，优化器会将 IR 转换为更高效的 IR。最后，LLVM 的后端会将 IR 转换为目标平台相关的汇编代码，并生成可执行文件。

在详细的架构图中，我们可以看到 LLVM 的前端、优化器、后端等各个组件的交互。在前端，Clang 会将高级语言代码转换为为 LLVM 的中间表示（IR）。

![编译器](../images/03Compiler01Tradition/04LLVM08.png)

在中间的编译优化层中，LLVM 会有非常多的 pass，不同的 pass 会对 IR 进行不同的优化处理，将 IR 转换为更高效的 IR。这里有关 pass 的详细内容将在下一节进行介绍。对 IR 的优化完成之后，编译器还是会以 LLVM 的中间表示（IR）作为输出，然后 LLVM 的后端会将 IR 转换为目标平台相关的机器码。

在后端中 LLVM 会进行指令选择，选择适合特定机器架构的指令。接下来，会进行寄存器分配，为指令分配合适的寄存器。然后，进行指令调度，优化指令执行顺序以提高性能。之后经过指令调度，会对代码进行布局，调整代码的排列顺序以适应目标硬件的执行特性。最后，会进行代码组装，生成目标硬件对应的汇编指令。

======= 有些内容过于口水话，应该更加描述得更加准确一点。如所以用户看到的就只是 Clang 的前端，而后面 LLVM 的优化器和后端都隐藏在幕后。藏在幕后？藏在哪里？架构很清楚，后端前端、中间层。

## LLVM 核心流程

编译器前端工作流程（词法、语法、语义分析）
中间优化层大数据中的 Pass 优化
编译器后端工作流程（机器指令选择、寄存器分配、指令调度）

======= 把上面改成一句话或者一段话哈，不要罗列出来。

![编译器](../images/03Compiler01Tradition/llvm_ir06.png)

在使用 LLVM 时，我们会从原始的 C 代码开始。这个 C 代码会经过一系列的预处理步骤，最终被转换为 LLVM 的中间表示文件（.ll 文件）或者 LLVM 字节码文件（.bc 文件）。

接下来使用 LLVM 的前端工具将中间表示文件编译成 IR。IR 的表示有两种方式，一种是 LLVM 汇编语言（.ll 文件），另一种是 LLVM 字节码（.bc 文件）。LLVM 汇编语言更为易读，方便人类阅读和理解。

IR 经过 LLVM 的后端编译器工具 llc 将 IR 转换为汇编代码（assembly code），这个汇编代码是特定机器（目标机器）的机器码指令的文本表示。

最后的两个步骤是将汇编代码汇编（assemble）成机器码文件，然后链接（link）生成可执行二进制文件，可以在特定平台上运行。

## 案例实践

Clang 编译流程。

======== 加一段话或者一句话，说明这下面或者里面 Clang 编译的是什么。 hello.c 有内容介绍吗？没有上下文哈。

1. 生成.i 文件

```shell
clang -E -c .\hello.c -o .\hello.i
```

2. 将预处理过后的.i 文件转化为.bc 文件

```shell
clang -emit-llvm .\hello.i -c -o .\hello.bc
```

```shell
clang -emit-llvm .\hello.c -S -o .\hello.ll
```

3. llc

======= 上面 1.2 都是具体的步骤，这里 3 是什么？

lld 链接器子项目旨在为 LLVM 开发一个内置的，平台独立的链接器[23]，去除对所有第三方链接器的依赖。在 2017 年 5 月，lld 已经支持 ELF、PE/COFF、 和 Mach-O。在 lld 支持不完全的情况下，用户可以使用其他项目，如 GNU ld 链接器。 lld 支持链接时优化。当 LLVM 链接时优化被启用时，LLVM 可以输出 bitcode 而不是本机代码，而本机代码生成由链接器优化处理。

```shell
llc .\hello.ll -o .\hello.s
llc .\hello.bc -o .\hello2.s
```

此时 hello.s 与 hello2.s 是相同的汇编代码。

4. 转变为可执行的二进制文件

```
clang .\hello.s -o hello
```

5. 查看编译过程

```
clang -ccc-print-phases .\hello.c
```

```shell
                +- 0: input, ".\hello.c", c
            +- 1: preprocessor, {0}, cpp-output
        +- 2: compiler, {1}, ir
    +- 3: backend, {2}, assembler
    +- 4: assembler, {3}, object
  +-5: linker, {4}, image
6: bind-arch,"x86_64", {5}, image
```

其中 0 是输入，1 是预处理，2 是编译，3 是后端优化，4 是产生汇编指令，5 是库链接，6 是生成可执行的 x86_64 二进制文件。

## 总结

- LLVM 把编译器移植到新语言只需要实现编译前端，复用已有的优化和后端；

- LLVM 实现不同组件隔离为单独程序库，易于在整个编译流水线中集成转换和优化 Pass；

- LLVM 作为实现各种静态和运行时编译语言的通用基础结构。

## 本节视频

<html>
<iframe src="https://player.bilibili.com/player.html?aid=817933285&bvid=BV1CG4y1V7Dn&cid=900870578&p=1&as_wide=1&high_quality=1&danmaku=0&t=30&autoplay=0" width="100%" height="500" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
</html>

## 引用

1. https://www.jetbrains.com/lp/devecosystem-2021/cpp/
2. https://zh.wikipedia.org/wiki/LLVM
3. https://github.com/llvm/llvm-project
4. https://bbs.huaweicloud.com/blogs/398739