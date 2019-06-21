# Chapter 4.2 - Filesystems

Created by : Mr Dk.

2019 / 06 / 21 13:54

@Nanjing, Jiangsu, China

---

对于磁盘来说，内核与用户空间之间的桥梁是文件系统

文件系统是一种数据库

包含能够将 block device 转变为不同等级的文件和目录的数据结构

Virtual File System (VFS) 抽象层完成了文件系统的实现

* 确保了所有文件系统的实现支持一个统一的接口
* 用户空间应用以相同的方式访问文件和目录

### 4.2.1 Filesystem Types

Linux 文件系统支持专为 Linux 优化的文件系统

也支持 Windows FAT 家族的其它文件系统

还支持 ISO 9660 等通用文件系统

* Forth Extended Filesystem (ext4) - Linux 默认的文件系统
  * ext2 是很长一段时间内 Linux 的默认系统
  * ext3 在 ext2 的基础上，在文件系统数据结构以外加入了一块缓存，加强数据完整性
  * ext4 文件系统在 ext3 的基础上，支持了更大的文件和更多数量的子目录
* ISO 9660 是一个 CD-ROM 标准
* FAT 文件系统 (msdos, vfat, umsdos) 
* HFS+ (hfsplus) - Apple 的文件系统

### 4.2.2 Creating a Filesystem

在用户空间中为设备创建文件系统

* 因为用户空间可以直接访问 block device

```bash
$ mkfs -t ext4 /dev/sdf2
```

Superblock 是文件系统数据库的核心组件

* `mkfs` 为 superblock 创建了多个备份，防止原先的那个被破坏

`mkfs` 只是一系列文件系统创建程序的前端

* `mkfs.fs` - `fs` 是文件系统类型
  * `mkfs.ext4` ...
  * `mkfs.ext4` 是 `mke2fs` 的符号链接

### 4.2.3 Mounting a FIlesystem

在 Unix 中，挂载一个文件系统的过程叫做 mounting

当系统启动时，内核读取一些配置信息，并基于配置信息挂载 root 根目录 (`/`)

为了挂载一个文件系统 - _mount a device on a mount point_

* 文件系统的设备 - 比如一个磁盘分区
* 文件系统的类型
* 挂载点 - 文件系统被关联到目前系统目录中的位置，是一个普通的目录

运行 `mount` 命令可以查看目前的文件系统状态

```bash
$ mount
rootfs on / type lxfs (rw,noatime)
none on /dev type tmpfs (rw,noatime,mode=755)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,noatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,noatime)
devpts on /dev/pts type devpts (rw,nosuid,noexec,noatime,gid=5,mode=620)
none on /run type tmpfs (rw,nosuid,noexec,noatime,mode=755)
none on /run/lock type tmpfs (rw,nosuid,nodev,noexec,noatime)
none on /run/shm type tmpfs (rw,nosuid,nodev,noatime)
none on /run/user type tmpfs (rw,nosuid,nodev,noexec,noatime,mode=755)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,noatime)
C: on /mnt/c type drvfs (rw,noatime,uid=1000,gid=1000,case=off)
D: on /mnt/d type drvfs (rw,noatime,uid=1000,gid=1000,case=off)
F: on /mnt/f type drvfs (rw,noatime,uid=1000,gid=1000,case=off)
```

每行的参数含义：

* 带有文件系统的设备（不一定是一个真实存在的设备）
* "on"
* 挂载点（一个目录）
* "type"
* 文件系统类型
* 挂载选项

为了挂载一个文件系统：

```bash
$ mount -t <fs_type> <device> <mount_point>
```

取消挂载：

```bash
$ umount <mount_point>
```

### 4.2.4 Filesystem UUID

挂载文件系统基于设备名

而设备名会根据内核找到设备的顺序而改变

因此，可以通过文件系统的 UUID (Universally Unique Identifier) 来挂载

* 是一种软件标准，是一种序列号
* `mke2fs` 等文件系统创建程序会在初始化文件系统数据结构时生成 UUID

可以通过 `blkid` 命令查看所有的设备、文件系统和 UUID：

```bash
$ blkid
/dev/vda1: UUID="59d9ca7b-4f39-4c0c-9334-c56c182076b5" TYPE="ext4"
```

那么，就可以使用 UUID 来挂载文件系统了：

```bash
$ mount UUID=<UUID> <mount_point>
```

### 4.2.5 Disk Buffering, Caching, and Filesystems

Linux 和其它 Unix 版本类似，会使用缓冲区写入磁盘

* 在处理变化请求时，内核不会立刻向文件系统写入变化
* 而是在 RAM 中存储变化，直到内核真正向磁盘写入变化
* 这一过程对用户透明

当 unmount 时，内核自动将缓冲区同步到磁盘

在其它时候，通过 `sync` 命令也可以使内核强制将修改写入磁盘

此外，内核有一系列机制使用 RAM 自动缓存从磁盘中读取的 disk

* 这样，如果一个或多个进程重复访问一个文件
* 内核不需要反复从磁盘中读取
* 节省时间和资源

### 4.2.6 Filesystem Mount Options

### 4.2.7 Remounting a Filesystem

### 4.2.8 The /etc/fstab Filesystem Table

为了在系统启动时挂载文件系统

Linux 系统在 `/etc/fstab` 中维护一个持久性的文件系统及其选项列表

```bash
$ cat /etc/fstab
UUID=59d9ca7b-4f39-4c0c-9334-c56c182076b5 /                       ext4    defaults        1 1
```

每一行代表一个文件系统，被分为 6 个部分：

* Device or UUID - 目前大部分 Linux 系统中已经开始使用 UUID
* 挂载点
* 文件系统类型
* 选项
* Backup information for use by the dump command
* 文件系统完整性测试顺序
  * `1` - root 文件系统
  * `2` - 其它文件系统
  * `0` - 禁止检查
    * 包括 swap、proc 文件系统

### 4.2.9 Alternatives to /etc/fstab

### 4.2.10 Filesystem Capacity

查看目前绑定的文件系统的 size 和利用率：

```bash
$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/vda1       41151808 4893668  34144708  13% /
devtmpfs          932488       0    932488   0% /dev
tmpfs             941860       0    941860   0% /dev/shm
tmpfs             941860     384    941476   1% /run
tmpfs             941860       0    941860   0% /sys/fs/cgroup
tmpfs             188376       0    188376   0% /run/user/0
```

* Filesystem - 文件系统设备
* 1K-blocks - 以 1KB 的块为单位的文件系统容量
* User - 已被使用的块数量
* Available - 空余的块数量
* Capacity - 正在被使用的块的百分比
* Mounted on - 挂载点

有一些空间被隐藏，作为 _reserved blocks_

只有超级用户可以使用所有的文件系统空间

* 这一特性可以防止系统服务在用尽磁盘空间后出错

`du` 命令可以从当前目录开始输出每个目录的磁盘使用量

`du -s` 只打印 summary

### 4.2.11 Checking and Repairing Filesystems

Unix 的文件系统优化是由专用的数据库机制完成的

内核必须相信已挂载的文件系统中没有任何错误

* 如果存在错误，那么将会发生数据丢失和系统崩溃

文件系统错误通常由用户强制关闭系统导致

* 在这种情况下，文件系统在内存中的缓存与磁盘不一致

`fsck` 是用于检查文件系统的工具

* 对于 `fsck` 有针对不同文件系统的不同版本

只需要给定设备文件或者挂载点：

```bash
$ fsck /dev/sdb1
```

`fsck` 会进行 5 个 passes：

* Pass 1: Checking inodes, blocks, and sizes
* Pass 2: Checking directory structure
* Pass 3: Checking directory connectivity
* Pass 4: Checking reference counts
* Pass 5: Checking group summary information ...

如果 `fsck` 发现了错误，会停止并询问是否要修复错误

* 修复改动的是文件系统的内部结构

仅对于 ext2 文件系统

对于 ext3、ext4 文件系统，不需手动检测

因为有机制能够保证文件系统的完整性

### 4.2.12 Special-Purpose Filesystems

不是所有的文件系统都代表了物理存储空间

大部分 Unix 版本包含了作为系统接口的文件系统

* 一些文件系统可以代表系统信息

Linux 中的专用文件系统包含：

* __proc__
  * 挂载于 `/proc` (process)
  * `/proc` 目录下的每一个数值目录都是系统中目前进程的 PID
  * `/proc/self` 代表了当前进程
  * Linux proc 文件系统包含了大量额外的内核和硬件信息，如 `/proc/cpuinfo`
* __sysfs__
  * 挂载于 `/sys`
* __tmpfs__
  * 挂载于 `/run` 和其它位置
  * 使用物理内存和交换空间作为暂存区

---

