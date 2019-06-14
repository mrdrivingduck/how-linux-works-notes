# Chapter 3 - Devices

Created by : Mr Dk.

2019 / 06 / 06 20:58

@Nanjing, Jiangsu, China

---

理解当内核发现新设备时，如何与用户空间进行交互

udev 系统使用户空间程序能够自动配置并使用新设备

* 内核如何通过 udev 向用户进程发消息？

---

## 3.1 Device Files

Unix 系统中将大部分设备的 I/O 接口表示为文件

这些设备文件有时被称为 _device nodes_

* 用户可以使用普通的文件操作使用设备
* 设备也可以被一些操作文件的标准程序使用 (cat 等)

但并不是所有设备都可以通过标准文件 I/O 来使用

在 Linux 中，设备文件位于 `/dev` 目录下

运行命令可以查看所有的设备文件：

```bash
[root@izuf6ewwd5zlnpjggtr0b4z ~]# ls -l /dev/
total 0
crw------- 1 root root     10, 235 Jan  5 23:56 autofs
drwxr-xr-x 2 root root          80 Jan  6 07:56 block
crw------- 1 root root     10, 234 Jan  5 23:56 btrfs-control
drwxr-xr-x 3 root root          60 Jan  5 23:56 bus
drwxr-xr-x 2 root root        2620 Jan  5 23:56 char
crw------- 1 root root      5,   1 Jan  5 23:56 console
lrwxrwxrwx 1 root root          11 Jan  6 07:56 core -> /proc/kcore
drwxr-xr-x 3 root root          80 Jan  5 23:56 cpu
crw------- 1 root root     10,  61 Jan  5 23:56 cpu_dma_latency
crw------- 1 root root     10,  62 Jan  5 23:56 crash
drwxr-xr-x 4 root root          80 Jan  6 07:56 disk
drwxr-xr-x 2 root root          80 Jan  6 07:56 dri
crw-rw---- 1 root video    29,   0 Jan  5 23:56 fb0
lrwxrwxrwx 1 root root          13 Jan  6 07:56 fd -> /proc/self/fd
crw-rw-rw- 1 root root      1,   7 Jan  5 23:56 full
crw-rw-rw- 1 root root     10, 229 Jan  5 23:56 fuse
crw------- 1 root root    249,   0 Jan  5 23:56 hidraw0
crw------- 1 root root     10, 228 Jan  5 23:56 hpet
drwxr-xr-x 2 root root           0 Jan  5 23:56 hugepages
crw------- 1 root root     10, 183 Jan  5 23:56 hwrng
lrwxrwxrwx 1 root root          25 Jan  5 23:56 initctl -> /run/systemd/initctl/fifo
drwxr-xr-x 4 root root         280 Jan  5 23:56 input
crw-r--r-- 1 root root      1,  11 Jan  5 23:56 kmsg
srw-rw-rw- 1 root root           0 Jan  6 07:56 log
crw-rw---- 1 root disk     10, 237 Jan  5 23:56 loop-control
drwxr-xr-x 2 root root          60 Jan  5 23:56 mapper
crw------- 1 root root     10, 227 Jan  5 23:56 mcelog
crw-r----- 1 root kmem      1,   1 Jan  5 23:56 mem
drwxrwxrwt 2 root root          40 Jan  5 23:56 mqueue
drwxr-xr-x 2 root root          60 Jan  5 23:56 net
crw------- 1 root root     10,  60 Jan  5 23:56 network_latency
crw------- 1 root root     10,  59 Jan  5 23:56 network_throughput
crw-rw-rw- 1 root root      1,   3 Jan  5 23:56 null
crw------- 1 root root     10, 144 Jan  5 23:56 nvram
crw------- 1 root root      1,  12 Jan  5 23:56 oldmem
crw-r----- 1 root kmem      1,   4 Jan  5 23:56 port
crw------- 1 root root    108,   0 Jan  5 23:56 ppp
crw-rw-rw- 1 root tty       5,   2 Jun 14 13:58 ptmx
drwxr-xr-x 2 root root           0 Jan  5 23:56 pts
crw-rw-rw- 1 root root      1,   8 Jan  5 23:56 random
drwxr-xr-x 2 root root          60 Jan  5 23:56 raw
lrwxrwxrwx 1 root root           4 Jan  5 23:56 rtc -> rtc0
crw------- 1 root root    253,   0 Jan  5 23:56 rtc0
drwxrwxrwt 2 root root          40 Jan  6 07:56 shm
crw------- 1 root root     10, 231 Jan  5 23:56 snapshot
drwxr-xr-x 2 root root          80 Jan  5 23:56 snd
lrwxrwxrwx 1 root root          15 Jan  6 07:56 stderr -> /proc/self/fd/2
lrwxrwxrwx 1 root root          15 Jan  6 07:56 stdin -> /proc/self/fd/0
lrwxrwxrwx 1 root root          15 Jan  6 07:56 stdout -> /proc/self/fd/1
brw-rw---- 1 root disk    253,   0 Jan  5 23:56 vda
brw-rw---- 1 root disk    253,   1 Jan  5 23:57 vda1
drwxr-xr-x 2 root root          60 Jan  5 23:56 vfio
crw------- 1 root root     10,  63 Jan  5 23:56 vga_arbiter
crw------- 1 root root     10, 137 Jan  5 23:56 vhci
crw------- 1 root root     10, 238 Jan  5 23:56 vhost-net
drwxr-xr-x 2 root root          60 Jan  6 07:56 virtio-ports
crw------- 1 root root    248,   1 Jan  5 23:56 vport1p1
crw-rw-rw- 1 root root      1,   5 Jan  5 23:56 zero
```

比如：

```bash
$ echo emm > /dev/null
```

将输出重定向到了一个文件中，而这个文件实际上对应了一个设备

由内核来决定如何处理写入该设备的数据

* 对于 `/dev/null` 来说，内核会忽略所有的输入，并将数据丢掉...... 😅

```bash
$ ls -l
brw-rw---- 1 root disk    253,   0 Jan  5 23:56 vda
crw------- 1 root root     10, 235 Jan  5 23:56 autofs
prw-r--r-- 1 root root           0 Mar  3 19:17 fdata
srw-rw-rw- 1 root root           0 Jan  6 07:56 log
```

File mode 的第一个字符如果是 `b` / `c` / `p` / `s`，那么该文件对应一个设备

* `b` - block device
* `c` - character device
* `p` - pipe device
* `s` - socket device

### Block Device

程序以固定大小的 chunks 从 block device 中访问数据

比如 __磁盘设备__

由于每个 chunk 大小固定，且易于索引

进程可以在内核的帮助下随机访问 block

### Character Device

这类设备的工作基于 __data streams__

只能对这类设备写入字符或读取字符

这类设备没有大小的概念

* 在进程中进行 read 或 write 的行为时，内核会通过驱动在设备上进行 read 或 write 操作
* 在数据被传输到设备或者进程之后，内核 __不会__ 备份或重新检验数据流

比如 __打印机__ 属于这类设备

### Pipe Device

这类设备和 character device 设备类似

但是 I/O stream 的另一端是另一个进程，而不是一个内核驱动

### Socket Device

用于进程间通信的特殊用途接口

通常在 `/dev` 目录之外

此外，前两行中日期之前的两个数字是设备的主要编号和次要编号

帮助内核识别设备

类似的设备通常都有类似的主要编号

__不是所有的设备都有设备文件，因为文件 I/O 接口不适用于所有的场景。__

## 3.2 The sysfs Device Path

`/dev` 目录中只有很少一部分的设备信息

Linux 内核通过 sysfs 接口，通过文件和目录

提供了查看设备真实硬件参数的统一视角

基路径位于 `/sys/devices`

* `/dev` 使用户进程能够使用设备
* `/sys/devices` 用于查看信息和管理设备
* 其中可能包含很多的符号链接，所以需要用 `ls -l` 来查看真实 sysfs 路径

在 `/sys/devices` 查找 `/dev` 中的路径可能很难

可以使用 `udevadm` 命令查找：（比如查找 `/dev/sda` 在 sysfs 下的路径）

```bash
$ udevadm info --query=all --name=/dev/sda
```

## 3.3 dd and Devices

`dd` 程序在 block devices 或 character devices 中非常有用

程序功能：

* 从输入流或输入文件中读取
* 写入输出文件或输出流
* 可能顺带做一些编码转换工作

`dd` 以固定块大小拷贝数据：

```bash
$ dd if=/dev/zero of=new_file bs=1024 count=1
```

* `if=file` - the input file
* `of=file` - the output file
* `bs=size` - the block size
  * `ibs=size`
  * `obs=size`
  * 用于输入输出的 block size 不同
* `count=num` - the total number of blocks to copy
* `skip=num` - 在输入文件或输入流中跳过前 num 个块，不拷贝到输出中

参数格式与其它 Unix 命令不同：IBM Job Control Language (JCL) style

