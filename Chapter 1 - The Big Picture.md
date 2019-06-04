# Chapter 1 - The Big Picture

Created by : Mr Dk.

2019 / 06 / 05 0:28

@Nanjing, Jiangsu, China

---

## 1.1 Levels and Layers of Abstraction in a Linux System

Linux 系统分为三个等级：

* Hardware
  * 主存、CPU、设备
* Kernel
  * 操作系统核心
* User Processes
  * 内核管理的运行程序

内核运行于 kernel mode

* 访问主存和 CPU 不受约束
* 强大且危险
* 内核进程可能使整个系统崩溃

用户进程运行于 user mode

* 受限访问一部分内存和安全的 CPU 操作
* 内核会清理用户进程的错误

## 1.2 Hardware: Understanding Main Memory

主存是一个存储 0 或 1 的巨大存储区

_state_ 经常被用于形容内存、进程或内核

* 其实就是特定排列的 bit 序列

注意：

_state_ 通常被用于形容 bits 的抽象排列

_image_ 通常被用于形容 bits 的物理排列

## 1.3 The Kernel

内核做的几乎所有事情都和主存有关

内核主要负责四个主要的管理工作：

* 进程管理 - 决定哪个进程可以使用 CPU
* 存储器管理 - 追踪所有的存储器，哪些被使用，哪些被共享，哪些被回收
* 设备驱动管理 - 硬件和进程的接口
* 系统调用和支持

### 1.3.1 Process Management

进程管理主要包含了进程的创建、暂停、继续、终结

在任何现代操作系统中，进程 __同时__ 运行

但实际上不是在相同的时刻被运行

想象一个单核 CPU

* 一次只能有一个进程使用 CPU
* 每个进程使用 CPU 一小段时间
* 进程将 CPU 控制权交给另一个进程的过程称为 __上下文切换 context switch__

每个时间片 - time slice - 给定进程的 CPU 使用时间

通常足够进程完成其目前的工作

然而，由于这个时间片非常小，人类难以察觉出来

从而产生了多个程序正被同时运行的错觉

内核负责上下文切换：

1. CPU 根据内部计时器中断目前的进程，切换到内核模式，将控制权交给内核
2. 内核记录目前的 CPU 和内存状态 - 用于之后的恢复
3. 内核处理上一时间片期间发生的 I/O 操作等
4. 内核就绪，分析进程的 ready list，并选择一个执行
5. 内核准备新进程的内存空间和 CPU
6. 内核告诉 CPU 该进程的时间片
7. 内核将 CPU 切换到 user mode，将 CPU 控制权交给进程

内核在两个时间片之间的时间内运行

### 1.3.2 Memory Management

内核的存储器管理极为复杂：

* 内核必须拥有用户空间不可访问的私有空间
* 每个用户进程需要有自己的内存区域
* 每个用户进程不能访问其它进程的私有内存
* 用户进程可以共享内存
* 用户进程部分内存只读
* 系统可以通过硬盘空间对换，使用比物理空间更多的内存

现代 CPU 通过 memory management unit (MMU) 引入了 virtual memory

使用虚存时，进程不直接访问物理内存

内核使每个进程感觉自己正在使用整个内存空间一样

MMU 将虚拟地址映射到实际的物理地址上

内核必须初始化并连续追踪内存映射的变化

内存映射表的实现叫做 page table - 页表

### 1.3.3 Device Drivers and Management

设备只能在 kernel mode 被访问

设备驱动属于内核的一部分

向用户进程提供统一的接口

### 1.3.4 System Calls and Support

两个重要的系统调用：

* `fork()` - 内核创建一个与源进程几乎相同的拷贝
* `exec()` - 内核执行程序，替换现有进程

shell 的实现流程

## 1.4 User Space

内核分配给用户进程的主存空间成为用户空间，也叫 userland

## 1.5 User

用户是运行进程和拥有文件的实体

每个用户关联一个 username

但内核通过唯一的 userids 识别用户

用户的存在主要为了权限和边界管理

每个用户空间进程都有一个 owner

* 进程被 run as owner

用户可以终结或修改自己拥有的进程行为

但不能干涉其它用户进程

root 用户 - superuser

* 终结或修改任意进程
* 读取任意文件

Groups 是 user 的集合，允许一个用户与组中其它用户共享文件访问权限

---

