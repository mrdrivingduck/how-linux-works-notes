# Chapter 4.1 - Partitioning Disk Devices

Created by : Mr Dk.

2019 / 06 / 15 17:16

@Nanjing, Jiangsu, China

---

Linux 磁盘的组织格式

分区 (Partitions) 是整个硬盘的 subdivisions

在 Linux 中，它们被表示为整体 block device 之后的编号

* 比如 `/dev/sda1`

内核将每一个分区视为一个 block device

分区表定义了硬盘上的分区，放置在硬盘上的小一块区域上

在每个分区上，可以运行文件系统

* 存放了文件和目录的数据

如果想要访问文件中的数据

* 在分区表中找到分区位置
* 查找分区上的文件系统数据库
* 访问文件

内核通过 block device interface 和 SCSI 子系统访问磁盘硬件

---

分区表的类型有很多种：

* _Master Boot Record, MBR_ - 传统分区表
* _Globally Unique Identifier Partition Table, GPT_ - 新标准

### 4.1.1 Viewing a Partition Table

```bash
$ parted -l
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 32.2GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
14      1049kB  5243kB  4194kB                     bios_grub
15      5243kB  116MB   111MB   fat32              boot, esp
 1      116MB   32.2GB  32.1GB  ext4
```

`parted` 程序所谓的 _msdos_ 就是传统的 MBR 分区表

在 GPT 中，每个分区有名字

MBR 表包含：

* Primary partitions - 主分区
* Extended partitions - 扩展分区
* Logical partitions - 逻辑分区

基本的 MBR 最多允许四个主分区

可以指定一个扩展分区，并将扩展分区划分为逻辑分区

#### Initial Kernel Read

使用 `dmesg` 命令可以看到：

```bash
$ dmesg
[    1.800660] sd 2:0:0:0: Power-on or device reset occurred
[    1.803385] sd 2:0:0:0: [sda] 62914560 512-byte logical blocks: (32.2 GB/30.0 GiB)
[    1.807989] sd 2:0:0:0: [sda] Write Protect is off
[    1.810075] sd 2:0:0:0: [sda] Mode Sense: 63 00 00 08
[    1.810281] sd 2:0:0:0: Attached scsi generic sg0 type 0
[    1.816186] sd 2:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
[    1.829643]  sda: sda1 sda14 < sda15 >
```

`sda14 < sda15 >` 表示 `/dev/sda14` 是一个扩展分区，包含 `/dev/sda15` 这个逻辑分区

### 4.1.2 Changing Partition Tables

* 改变分区表很难恢复分区上的数据
  * 因为改变了文件系统引用的起始点
* 保证目标磁盘上没有分区正在被使用
  * 因为大部分 Linux 发行版会自动绑定任何检测到的文件系统

### 4.1.3 Disk and Partition Geometry

磁盘包含一根轴 _Spindle_ 和一组重叠的旋转盘片 _Platter_

以及位于一个可移动的手臂 _Arm_ 上的磁头 _Head_

磁头用于读取数据

当手臂在某个位置时，磁头只能读取一个固定的圆柱面上读取数据

* 这个圆柱面被叫做柱面 _Cylinder_

每个盘片可以有一或两个磁头

* 连接到同一个手臂上
* 分别操作盘片的上下两面

磁盘上有很多柱面，越往外围越大

每个柱面被分为很多个片，称为扇区 _Sector_

所谓的磁盘 CHS 即 cylinder-head-sector

内核和分区程序报告的硬盘柱面、扇区数据是假的

* 在现代磁盘中不适用
* 磁盘硬件通过 Logical Block Addressing, LBA 寻找磁盘上的位置

柱面是理想化的边界

读取同一柱面上的数据非常快，因为磁头不需要移动

读取相邻柱面上的数据也很快，因为磁头不需要移动很远

### 4.1.4 Solid-State Disks (SSDs)

内部没有移动的部分

每次以块 (typically 4096B) 为单位读取数据

读取必须从块大小的整数倍的位置开始读取

如果数据跨过了块大小边界，则需要多次读取

很多分区工具保证了在磁盘起始位置的合适偏移量处放置新分区

用户不需要关心分区组织

---

