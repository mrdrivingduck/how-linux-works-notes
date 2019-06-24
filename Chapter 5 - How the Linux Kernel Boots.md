# Chapter 5 - How the Linux Kernel Boots

Created by : Mr Dk.

2019 / 06 / 25 0:14

@dorm, NUAA, Nanjing, Jiangsu, China

---

内核如何启动？

1. BIOS 或固件装载并运行 boot loader
2. boot loader 找到磁盘上的内核映像，将其装入内存并启动
3. 内核初始化设备及其驱动
4. 内核挂载根文件系统
5. 内核启动 PID 为 1 的程序 `init` - 此时用户空间启动
6. `init` 启动剩余的系统进程
7. `init` 启动一个用于登录的进程

---

## 5.1 Startup Messages

传统的 Unix 系统会打印大量表示启动过程的信息

但现代系统已经尽可能隐藏这些信息了

如果想看，可以：

* 查看内核日志文件 - `/var/log/kern.log`
* 使用 `dmesg` 命令，但需要通过管道连接到 `less` 中，否则太多

---

## 5.2 Kernel Initialization and Boot Options

Linux 内核的初始化顺序：

1. CPU 检查
2. 存储器检查
3. 发现设备总线
4. 发现设备
5. 内核辅助子系统启动
6. 根文件系统挂载
7. 用户空间启动

---

## 5.3 Kernel Parameters

在运行 Linux 内核时

boot loader 传递一系列文本参数

告诉内核它应该如何启动 - _kernel parameters_

可以在 `/proc/cmdline` 中查看内核参数

```bash
$ cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-4.15.0-51-generic root=UUID=af284c6e-d36f-410d-8c9e-950532e5ee89 ro console=tty1 console=ttyS0
```

* one-word flags
* key=value pairs

其中，`root` 代表根文件系统所在位置

* 可以是一个设备文件（分区）
* 现在桌面系统中，UUID 更加流行

`ro` 命令内核以 read-only 的方式挂载根文件系统

* 确保 `fsck` 能够安全地检测根文件系统
* 检测完成之后，内核重新以 read-write 模式挂载根文件系统

对于内核不认识的参数

内核会将其作为 `init` 函数的参数传递过去

---

## 5.4 Boot Loaders

在启动的过程中

由 boot loader 来启动内核

功能看起来很简单：

* 将内核装载到内存中，将一些列 kernel parameter 传递给内核，并启动内核

但是 boot loader 必须能够回答以下问题：

* 内核在哪里？
* 当内核启动时，应当将哪些参数传递给它

显然，内核及其参数肯定位于根文件系统的某个位置

然而，此时内核还没有开始运行

因此无法通过遍历文件系统来寻找

此外，用于访问磁盘的设备驱动也无法使用

__chicken or egg__ problem

在 PC 上，boot loader 使用 - 

* __Basic Input/Output System (BIOS)__
* __Unified Extensible Firmware Interface (UEFI)__

来访问磁盘

* 几乎所有的磁盘硬件都有允许 BIOS 使用 _Linear Block Addressing (LBA)_ 来访问存储的固件
* 这种访问方式性能很低
* 但允许磁盘的完全访问
* Boot loader 通常是使用这种访问方式的唯一程序
* 内核使用其高性能的驱动程序访问磁盘

有一些先进的 boot loader 能够读取分区表

并提供了内置以 read-only 的方式访问文件系统的支持

因此 boot loader 可以寻找并读取文件

### 5.4.1 Boot Loader Tasks

一个 Linux boot loader 的核心功能：

* 在多个内核中选择
* 在一系列内核参数中切换
* 允许用户手动覆盖或编辑内核映像名和参数
* 提供启动其它操作系统的支持

### 5.4.2 Boot Loader Overview

* GRUB - Linux 系统的基本标配
* LILO - 最早期的 Linux boot loader 之一
* SYSLINUX
* LOADLIN - 从 MS-DOS 启动内核
* efilinux
* coreboot
* Linux Kernel EFISTUB - 内核插件，直接从 ESP 分区装载内核

---

## 5.5 GRUB Introduction

GRUB - _Grand Unified Boot Loader_

GRUB 最重要的能力是文件系统导航

* 使得选择内核映像和配置更加简单（从 GRUB 的菜单就可以看出）

当 BIOS 或固件启动时，按住 shift 可以进入 GRUB 菜单

在菜单界面超时后，会自动启动

按下 `e` 按钮可以查看 boot loader 的配置命令

```
setparams 'Ubuntu, with Linux 3.2.0-31-generic-pae'
recordfail
gfxmode $linux_gfx_mode
insmod gzio
insmod part_msdos
insmod ext2
set root='(hd0,msdos1)'
search --no-floppy --fs-uuid --set=root 4898e145-b064-45hd-b7b4-7326b00273b7
linux /boot/vmlinuz-3.2.0-31-generic-pae root=UUID=489e145-b064-45bd-b7b4-7326b00273b7 ro quiet splash $vt_handoff
initrd /boot/initrd.img-3.2.0-31-generic-pae
```

* 内核映像为 `/boot/vmlinuz-3.2.0-31-generic-pae`
* 根文件系统由 UUID 标识
* 内核参数包含 `ro`、`quiet`、`splash`
* 初始化 RAM 文件系统为 `/boot/initrd.img-3.2.0-31-generic-pae`

GRUB 只用于启动内核，因此不依赖于内核

上述命令全部是 GRUB 的内部命令

* GRUB 有自己的内核
* GRUB 有自己的 `insmod` 命令，用于动态载入 GRUB 模块
* 很多 GRUB 命令和 Unix shell 的命令类似，比如 `ls`

上述命令中，只有 linux 命令下的 root 表示根文件系统的位置

其它 root 是 GRUB 的 root

* 是 GRUB 搜索内核和 RAM 文件系统映像的位置

在上述命令中，`root` 首先被设定为 `hd0,msdos1`

并在之后指定了一个 UUID，GRUB 在分区上查找该 UUID

如果找到，则将 `root` 设定为该 UUID 所在分区

`linux` 和 `initrd` 命令表示从 GRUB root 所在分区中的 `/boot` 目录下载入映像

以上配置命令可以在 GRUB 中修改

### 5.5.1 Exploring Devices and Partitions with the GRUB Command Line

通过按下 `C` 可以进入 GRUB 命令行

#### Listing Devices

```
grub> ls
(hd0)  (hd0,msdos1)  (hd0,msdos5)
```

* `ls` 命令列出了 GRUB 识别的所有设备
* `msdos` 表示磁盘包含 MBR 分区表，因此可能会有 `gpt` 的前缀

使用 `ls -l` 可以详细查看各个分区的信息

* 文件系统类型
* 大小、最后修改时间
* UUID

#### File Navigation

GRUB 的文件系统导航能力

```
grub> echo $root
hd0,msdos1
```

```
grub> ls (hd0,msdos1)/
```

```
grub> ls ($root)/
```

```
grub> ls ($root)/boot
```

使用 `set` 命令可以查看所有的 GRUB 变量：

```
grub> set
?=0
color_highlight=black/white
color_normal=white/black
--snip--
prefix=(hd0,msdos1)/boot/grub
root=hd0,msdos1
```

* `$prefix` 是 GRUB 寻找其配置文件和额外支持的路径

### 5.5.2 GRUB Configuration

GRUB 的配置目录包含一个核心配置文件 `grub.cfg` 和大量可装载的模块 `*.mod`

#### Reviewing Grub.cfg

#### Generating a New Configuration File

### 5.5.3 GRUB Installation

#### Installing GRUB on Your System

安装 boot loader 需要用户或安装器决定：

* GRUB 的目标路径，默认为 `/boot/grub`，但其它操作系统不同
* GRUB 安装的目标设备
* 对于 UEFI 启动，还需要 UEFI boot 分区的挂载点

GRUB 是一个模块化的系统，需要读取文件系统下的 GRUB 路径

所以在 Linux 上，只需要将 GRUB 和 ext2.mod 预装载版本放置在磁盘的启动区

其余部分放置在 GRUB 的目录下即可

---

## 5.6 UEFI Secure Boot Problems

UEFI 的安全机制要求 boot loader 必须被可信方签名后才可运行

* 无法安装未被签名的 boot loader

---

## 5.7 Chainloading Other Operating System

UEFI 使得装载其它操作系统变得简单

因为可以在 EFI 分区中安装多个 boot loader

通过 chainloading，可以使 GRUB 装载并运行一个指定磁盘分区上的另一个 boot loader

---

## 5.8 Boot Loader Details

两种主要的启动模式：

* MBR
* UEFI

### 5.8.1 MBR Boot

_Master Boot Record (MBR)_ 除了包含分区信息以外

还包括一个 `441` Bytes 的区域

用于 BIOS 在 _Power-On Self-Test (POST)_ 之后装载并执行

但由于这个区域对于 boot loader 来说太小

需要一个额外的空间

即所谓的 _multi-stage boot loader_

此时，MBR 的初始空间只用于装载 boot loader 的剩余代码

剩余部分的 boot loader 代码通常位于 MBR 和磁盘第一个分区之间

### 5.8.2 UEFI Boot

PC 制造商和软件公司意识到传统的 BIOS 有诸多局限

因此开发了替代品 - _Extensible Firmware Interface (EFI)_

目前的标准是 _Unified EFI (UEFI)_

* 包括一个内置的 shell
* 和读取分区表，导航文件系统的能力

GPT 分区模式是 UEFI 标准的一部分

总是会有一个特殊的文件系统 - _EFI System Partition (ESP)_

* 包含一个名为 `efi` 的目录
* 每个 boot loader 有其自身的标识和对应的子目录
* `.efi` 的 boot loader 文件分别存放在每个子目录中

BIOS 和 UEFI 的 boot loader 版本是不同的

### 5.8.3 How GRUB Works

1. BIOS/固件初始化硬件，搜索带有 boot code 的存储设备
2. BIOS/固件载入并执行 boot code - GRUB begins
3. GRUB core 装载
4. GRUB core 初始化，现在 GRUB 可以开始访问磁盘和文件系统
5. GRUB 识别其 boot 分区，加载配置文件
6. GRUB 给用户提供修改配置的机会
7. 在用户选择或超时后，GRUB 执行当前配置
8. GRUB 在 boot 分区中加载其它代码 (modules)
9. GRUB 执行 boot 命令，装载并执行内核

问题：__Where is the GRUB core?__

* 除了 ESP 以外，BIOS 读取 MBR 的 `512` 字节，即 GRUB 开始的地方
  * 不包含 GRUB core
  * 但包含 GRUB 的起始位置，和从该位置加载的代码
* 如果使用 ESP，则固件可以直接寻找到 GRUB core 的文件并执行

---

