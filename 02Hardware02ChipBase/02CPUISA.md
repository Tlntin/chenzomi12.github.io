<!--Copyright 适用于[License](https://github.com/chenzomi12/AISystem)版权许可-->

# CPU 指令集架构

我们知道，计算机指令是指挥机器工作的指示和命令，程序就是一系列指令按照顺序排列的集合，执行程序的过程就是计算机的工作过程。从微观上看，我们输入指令的时候，计算机会将指令转换成二进制码存储在存储单元里面，然后在即将执行的时候拿出来。那么计算机是怎么知道我们输入的是什么指令，指令要怎么执行呢？

这就要提到 ISA 也就是指令集架构，本节内容主要围绕 CPU 的指令集架构 ISA 展开，学习 CISC 架构与 RISC 架构并并对比两种架构的优劣势，进而介绍相关的应用场景。

## ISA 指令集架构

通常用来区分 CPU 的标准是指令集架构（instruction set architecture，ISA），简称 ISA。下面将会通过例子介绍 ISA 如何运作，ISA 的作用以及 ISA 的分类和生命周期。

### 什么是 ISA

ISA 是处理器支持的所有指令的语义，包括指令本身及其操作数的语义，以及与外围设备的接口。就像任何语言都有有限的单词一样，处理器可以支持的基本指令/基本命令的数量也必须是有限的，这组指令通常称为指令集（instruction set），基本指令的一些示例是加法、减法、乘法、逻辑或和逻辑非。

开发人员基于指令集架构（ISA），使用不同的处理器硬件实现方案，来设计不同性能的处理器，因此 ISA 又被视作 CPU 的灵魂。

指令集架构是软件感知硬件的方式，我们可以将其视为硬件输出到外部世界的基本功能列表。Intel 和 AMD CPU 使用 x86 指令集，IBM 处理器使用 PowerPC R 指令集，HP 处理器使用 PA-RISC 指令集，ARM 处理器使用 ARMR 指令集（或其变体，如 Thumb-1 和 Thumb-2）。

因此，不可能在基于 ARM 的系统上运行为 Intel 系统编译的二进制文件，因为指令集不兼容，但在大多数情况下，可以重用 C/C++程序。要在特定架构上运行 C/C++程序，我们需要为该特定架构购买一个编译器，然后适当地编译 C/C++程序。

我们从更宏观的视角来看看指令集架构，可以将指令集架构理解为一个抽象层，它是处理器底层硬件与运行在硬件上的软件之间桥梁和接口，如下层所示上面是软件部分，下面是硬件部分。

![ISA](../images/02Hardware02ChipBase/cpu/ISA01.png)

这也就可以解释计算机是怎么知道我们输入的是什么指令，指令要怎么执行呢？

计算机就可以通过指令集，判断这一段二进制码是什么意思，然后通过 CPU 转换成控制硬件执行的信号，从而完成整个操作，这样一来，指令集其实就是硬件和软件之间的接口（interface），我们不再需要直接和硬件进行交互，而是和具有更高的抽象程度的 ISA 进行交互，集中注意在指令的编写逻辑，提高工作效率。

### ISA 历史种类

====== PPT 提到了的内容，文章不能少，应该更多详细的展开。

### Add 指令例子

接下来我们把 ISA 其中的一个指令拿出来详细看看指令的组成，如下图所示是一个 MIPS 指令，这是一种采取精简指令集（RISC）的处理器架构。

> 精简指令集（RISC）的处理器架构：1981 年出现，由 MIPS 科技公司开发并授权，广泛被使用在许多电子产品、网络设备、个人娱乐设备与商业设备上。最早的 MIPS 架构是 32 位，最新的版本已经变成 64 位。

例子中的 Add 指令共分为两个部分，首先左边的第一个部分就是运算符，需要告诉我们它的 Op Code 是多少，对应了不同的运算符也就是不同的操作，图中是以“加法”为例。之后右边三个部分对应着的是我们的操作对象，其中一共有三个参数分别是：目的操作数 Addr1、原操作数 Addr2 以及立即数 imediate Value。

指令的含义也就是将立即数与原操作数相加存储到目的操作数之中。

  ![ISA](../images/02Hardware02ChipBase/cpu/ISA03.png)

====== 这里例子介绍得不够详细，网上有很多，建议多摘录。

### ISA 基本分类

对于指令集的指令分类主要有三种，分别是：

1. 运算指令：在 ALU 中执行的计算操作

2. 数据移动指令：读写存储操作（包括寄存器读写）

3. 控制指令：更改指令执行顺序，进行程序跳转，实现 if/else，循环等

对于不同厂家的 GPU 而言，都会有自己独特的运算指令来做一些特殊的操作，指令编写的好坏直接影响着 GPU 的计算性能。指令集的强弱也是芯片的重要指标，指令集是提高处理器效率的最有效工具之一。采用相同架构的处理器，性能基本上已经锁定在一定的范围之内，不会有本质的区别。

======= 过于简单没有展开

### ISA 生命周期

接下来我们简单介绍一下 ISA 的生命周期，主要分为如下所示的 6 个阶段，虽然不是所有指令都会循环所有阶段，但基本上大原则都不会变。

1. FETCH：将 Memory 中的指令放入 Instruction Register ，PC 指定指令位置
2. DECODE：通过指令解码过程，识别指令内容，开启控制信号通路
3. EVALUATE ADDRESS：比如加载内存之前，通过寄存器内容和偏移量获得真正的内存位置
4. FETCH OPERANDS：加载寄存器中的操作数，不同指令实际执行的内容不同
5. EXECUTE：在 ALU 中执行计算逻辑
6. STORE RESULT：存储计算结果

======= 过于简单没有展开

## CISC vs RISC 

目前看来，按照指令系统复杂程度的不同，CPU 的 ISA 可分为 CISC 和 RISC 两大阵营。CISC 是指复杂指令系统计算机（Complex Instruction Set Computer）；RISC 是指精简指令系统计算机（Reduced Instruction Set Computer），如下图所示也是很好区分 RISC 和 CISC。

 ![ISA](../images/02Hardware02ChipBase/cpu/ISA02.png)

### CISC 架构

除常用指令还包含许多不常用特殊指令。随着越来越多的特殊指令被添加到 CISC 架构中，常用程序运算指令仅占指令集 20%，80% 指令则很少用到，而这些很少用到的指令让 CPU 的设计变得极其复杂，大大增加了硬件设计的时间成本和面积开销。

======= 过于简单没有展开

### RISC 架构

只包含处理器常用指令，对于不常用操作，执行多条常用指令的方式来达到同样的效果。因而在 RISC 架构诞生后，移动端设备设计的 CPU 都倾向于选择使用 RISC 架构。

======= 过于简单没有展开

## CPU 应用场景

======= 面向不同的应用场景，使用不同的指令集架构，应该展开阐述自己的理解和目前实际业界的状况。

## 总结

CPU 具备图灵完备，可自运行的处理器。其主动从指令内存读取指令流，然后译码后执行；指令执行会涉及到数据的载入(Load)、计算和存储(Store)。而 ISA 是计算机体系结构与编程相关的部分，定义了：指令集、数据类型、寄存器、寻址模式、内存管理、I/O 模型等。总的来说，可以把 CPU 简单地分为控制平面和计算平面两部分，CPU 作为指令流驱动计算的处理引擎。

## 本节视频

<html>
<iframe src="https://player.bilibili.com/player.html?aid=396874348&bvid=BV1ro4y1W7xN&cid=1101969636&page=1&as_wide=1&high_quality=1&danmaku=0&t=30&autoplay=0" width="100%" height="500" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
</html>