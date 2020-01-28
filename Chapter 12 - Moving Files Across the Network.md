# Chapter 12 - Moving Files Across the Network

Created by : Mr Dk.

2019 / 07 / 12 14:33

@NUAA, Nanjing, Jiangsu, China

---

除了 `scp` 和 `sftp` 以外的文件共享方式。

## 12.1 Quick Copy

如果只是为了快速，到包含想拷贝文件的目录中，执行：

```bash
python -m SimpleHTTPServer
```

在该目录下开启了一个 web 服务器，默认使用 `8000` 端口。远程机器用浏览器访问该机器的 `8000` 端口即可访问该目录。

## 12.2 rsync

使用 `scp -r` 可能不会获得某个远程目录的完整拷贝。在 Linux 中，rsync 是一个专用的同步系统。

### 12.2.1 rsync Basics

为了能够工作，rsync 必须运行在两台机器上同时运行，还需要两台机器能够互相访问，比如通过 SSH。现在的系统中，rsync 默认使用 SSH 来连接远程主机：

```bash
rsync <file1> <file2> ... user@host:<dest_dir>
```

rsync 默认只拷贝文件。为了拷贝目录中的完整信息，包括符号链接、权限、设备等，使用 `-a` 选项：

```bash
rsync -a <dir> user@host:<dest_dir>
```

为了防止出错，可以加入 `-n` 选项，进行一次 dry run 模拟。`-v` 选项加入了输出信息。

### 12.2.2 Making Exact Copies of a Directory Structure

默认情况下，rsync 不关心目标目录中的已有文件。假设目标目录已有文件，将会与拷贝文件共存。为了在拷贝过程中保持与源目录完全相同，使用 `--delete` 选项将目标目录中的已有文件删除：

```bash
rsync -a --delete <dir> user@host:<dest_dir>
```

### 12.2.3 Using the Trailing Slash

在 rsync 中拷贝目录时需要格外小心：

```bash
rsync -a dir user@host:<dest_dir>
rsync -a dir/ user@host:<dest_dir>
```

两条命令的行为是不一样的：

- 第一条命令的行为是拷贝目录
  - 如果 `dir` 下有 `dir/a` 文件
  - 结果应当是 `/dest_dir/dir/a`
- 第二条命令的行为是拷贝 `dir` 路径下的所有文件
  - 结果应当是 `/dest_dir/a`

### 12.2.4 Excluding Files and Directories

```bash
rsync -a --exclude=.git <dir> user@host:<dest_dir>
```

`--exclude` 接受的是 pattern，因此会忽略所有的 `.git`。如果只是想忽略某一个 `.git`，就需要使用绝对路径：

```bash
rsync -a --exclude=/src/.git <dir> user@host:<dest_dir>
```

`/src/.git` 中的第一个 `/` 并不是根目录，而是传输的基目录。

### 12.2.5 Transfer Integrity, Safeguards, and Verbose Modes

为了加速操作，rsync 使用快速检查来决定传输的文件是否已经在目标路径下：基于 **文件大小** + **最后修改日期** 的快速判断。第一次使用 rsync 时，由于所有文件都不存在，因此 rsync 会传输所有文件。如果文件已经存在，rsync 则不会再次传输。对于不同步的文件，rsync 将自动覆盖。

- `--checksum` / `-c`：计算文件的校验和，判断两个文件是否一致，需要额外占用 I/O 和 CPU，但对于敏感数据是值得的
- `--ignore-existing`：不覆盖目标路径中的已有文件，忽略
- `--backup` / `-b`：不覆盖目标路径中的已有文件，在传输文件前，先将已有文件重命名为 `~` 后缀
- `--suffix=<...>`：修改 `--backup` 选项的重命名后缀
- `--update` / `-u`：如果已有文件的修改日期比源文件晚，则禁止覆盖

### 12.2.6 Compression

在传输之间压缩数据：`-z` 选项：

```bash
rsync -az <dir> user@host:<dest_dir>
```

对于数据量较大，或者网络延时较高的情况，用 `-z` 可以提升性能。但是对于高速局域网来说，没有必要花费额外的 CPU 时间来进行压缩 / 解压缩。

### 12.2.7 Limiting Bandwidth

限制带宽为 `10000` Kpbs：

```bash
rsync --bwlimit=10000 -a <dir> user@host:<dest_dir>
```

### 12.2.8 Transferring Files to Your Computer

与 `scp` 类似，可以双向拷贝：

```bash
rsync -a user@host:<src_dir> <dest_dir>
```

### 12.2.9 Further rsync Topics

可以使用 `rsync --delete` 周期性地同步文件系统。

