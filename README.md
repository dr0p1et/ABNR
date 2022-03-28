# ·天书夜读·专栏

这个专栏主要用于记录自己平时的读书笔记和思维碎片。

Study notes can persist for three years, you will be able to become a teacher.

## 思维碎片

[Pieces of Mind.](./pieces_of_mind.md)

## 我的书单

[Book List.](./books_list.md)

## 阅读计划

- 第1期：Linux内核学习路线（上）
  - 操作系统设计与实现 Minix3
  - Linux内核设计与实现
  - 深入理解Linux内核
  - Linux内核源代码完全剖析
  - 深入Linux内核架构
- 第2期：Linux内核学习路线（下）
  - 深入理解Linux虚拟内存管理
  - 深入理解Linux网络技术内幕
  - 存储技术原理分析
  - Linux设备驱动程序（第三版）
  - 鸿蒙内核剖析百篇
  - 深入浅出系统虚拟化原理与实现
- 第3期：Intel/AMD开发手册导读
  - Intel® 64 and IA-32 Architectures Software Developer's Manual
  - AMD64 Architecture Programmer's Manual
- 第x期：零散阅读计划
  - 只是为了好玩 Linus自传
  - 暂无

### 第1-2期：Linux内核学习路线

共分两期，2022年至2023年两年的学习计划，目标是达到Linux内核高级工程师的工作要求。

![](./notes/linux_kernel_study_path.png)

#### 第1期

2022年底完成前奏、基础、入门和高级内容的学习，包括五本书和两套课，通读Linux 0.12内核代码，使自己对内核框架能有清晰的认识。

年底任务：

- 编写一款精简的64位操作系统RainOS。以Linux 0.12和Minix3内核代码为基础，集成Linux 2.6高版本优秀特性，尽可能简化内核模块，删除不需要的低版本陈旧功能，保留最优化的功能代码，包括进程、内存、IO、文件、网络五个部分，实现最基本的shell交互，最终在x86开发板上运行起来。
- 完成《八小时从零默写一个操作系统》冰桶挑战。

#### 第2期

2023年底完成主要子系统（进程、内存、IO、文件、网络、安全子系统）、微内核（主要是鸿蒙和seL4）和虚拟化（kvm和docker三大件）的学习，包括各个子系统的书籍和源码实现、通读鸿蒙和seL4内核源代码、通读kvm和docker源代码等内容。

年底任务：

- 华为云计算HCIE和鸿蒙内核工程师考证。
- 自己动手写虚拟机—精简的kvm。
- 内核提权和虚拟机逃逸的Fuzzing工具。

### 第3期：Intel/AMD开发手册导读

长期计划，完成Intel和AMD开发手册的阅读。

- Intel® 64 and IA-32 Architectures Software Developer's Manual Volume 1-3
- AMD64 Architecture Programmer's Manual Volume 1-5

参考国内出版的《x86/x64体系探索及编程》和《处理器虚拟化技术》两本书。

任务：

- 中文版导读手册。

以导读翻译的形式出一本导读手册，以雅的翻译手法从整体上把握开发手册的核心要义。

### 第x期：零散阅读计划

暂无。

---

:copyright: 2021-2022 :rocket: `DR0p1ET`.
