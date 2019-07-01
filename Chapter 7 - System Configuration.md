# Chapter 7 - System Configuration: Logging, System Time, Batch Jobs, and Users

Created by : Mr Dk.

2019 / 07 / 01 21:25

@NUAA, Nanjing, Jiangsu, China

---

## 7.1 The Structure of /etc

大部分的配置文件存放在 `/etc` 下的子目录中

* 针对单机的自定义配置文件，比如：
  * 用户信息 - `/etc/passwd`
  * 网络细节 - `/etc/network`
* 通用的应用信息不存放在 `/etc` 下

## 7.2 System Logging

大部分系统程序会将其诊断信息输出到 `syslog` 服务上

* `syslog` 等待消息输入
* 根据接受到消息的类型，将其输出到文件、屏幕、用户，或简单忽略

### 7.2.1 The System Logger

`rsyslogd` - new version of `syslogd`

在 `/var/log` 中有很多日志信息

但很多文件不是由系统日志维护的

### 7.2.2 Configuration Files

`rsyslogd` 的基本配置文件位于 `/etc/rsyslog.conf`

* 其中包含了多种规则
* 规则包含：
  * selector - how to catch logs
  * action - where to send them

selector 表示记录日志的信息类型，位于左侧

action 表示日志被输出到何处，位于右侧

```
kern.*                  /dev/console
*.info;authpriv.none    /var/log/messages
authpriv.*              /var/log/secure,root
mail.*                  /var/log/maillog
cron.*                  /var/log/cron
*.emerg                 *
local7.*                /var/log/boot.log
```

#### Facility and Priority

selector 需要匹配日志消息的 facility 和 priority

* Facility
  * 消息中包含何种信息，比如包含 `kern`、`mail` 等
* Priority
  * 在 facility 和 `.` 之后
  * Order (from lowest to highest):
    * debug
    * info
    * notice
    * warning
    * err
    * crit
    * alert/emerg
  * 使用 `.none` 的优先级可以除去某个 facility

#### Extended Syntax

rsyslogd 继承了 syslogd 的语法

配置文件的扩展叫做 _directives_，通常由 `$` 开头

可以用于载入额外的配置文件

#### Troubleshooting

使用 `logger` 命令可以注入日志

## 7.3 User Management Files

Unix 系统允许多个独立用户存在

在内核层面，用户被简单地表示为 _user ID_

为了方便记忆，在用户空间会被表示为 _username_

### 7.3.1 The /etc/passwd File

明文的 `/etc/passwd` 将 usernames 和 user IDs 进行映射

```bash
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:*:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
mrdrivingduck:x:1000:1000:,,,:/home/mrdrivingduck:/bin/bash
```

每行的内容分别为：

* Username
* 加密后的用户密码
  * 密码本身存储在 _shadow_ 文件中 - `/etc/shadow`
    * 普通用户没有对该文件的读权限
    * 密码以加密形式保存在 shadow 文件中 - Unix 密码永远不会明文存储
  * `x` 表示加密的密码存储在 shadow 文件中
  * `*` 表示用户无法登陆
  * ` ` 则表示登陆无需密码
* user ID (UID)
* group ID (GID)
* 用户的 real name
* 用户的主目录
* 用户的 shell

该文件中不允许注释和空行

### 7.3.2 Special Users

superuser - root 的 UID 和 GID 总为 0

有一些用户是没有登陆权限的

_nobody_ 用户是无特权用户，将无法向系统写入任何数据

不能登陆的用户被称作 _pseudo-users_

* 系统能够以它们的 UID 启动进程
* 通常用于安全性原因而启动

### 7.3.3 The /etc/shadow File

### 7.3.4 Manipulating Users and Passwords

使用 `passwd` 命令与 `/etc/passwd` 进行交互，修改用户的密码

* `-f` 用于改变用户的 real name
* `-s` 用于改变用户的 shell
* 是一个 suid-root 程序，只有 root 用户才能修改 `/etc/passwd`

### 7.3.5 Working with Groups

Unix 通过 groups 来实现特定用户之间的文件共享

* 在多用户共享一台主机时相当重要

在 `/etc/group` 文件中定义了 group ID

```
root:*:0:juser
daemon:*:1:
bin:*:2:
user:*:1000:
```

每行的内容为：

* group name
* group password - 很少用到
* group ID - 必须是唯一的，并出现在 `/etc/passwd` 中的 GID 中
* 属于该 group 的可选用户

Linux 通常为新用户创建一个同名的 group

## 7.4 getty and login

`getty` 是绑定 terminal 并显示登录界面的程序

```bash
$ ps ao args | grep getty
/sbin/agetty --noclear tty1 linux
/sbin/agetty --keep-baud 115200 38400 9600 ttyS0 vt220
```

其中的数字为波特率，只是为了向后兼容，现在已经很少使用

在输入登录的用户名后，`getty` 将自身替换为 `login` 程序

如果密码正确，`login` 程序再将自身替换为 shell 程序 (通过 `exec()`)

但目前的登录主要通过：

* 图形接口 - `gdm`
* 远程 - SSH

## 7.5 Setting the Time

内核维护系统时钟

PC 硬件中包含一个电池供电的 _real-time clock (RTC)_

内核通常在启动时将其时间设置为与 RTC 一致

可以通过 `hwclock` 命令将目前的系统时间复位为 RTC 硬件时间

也可以将系统的时间设置到 RTC 中

### 7.5.1 Kernel Time Representation and Time Zones

内核的系统时钟将目前时间表示为 1970.1.1 00:00 到现在的秒数：

```bash
$ date +%s
1561984433
```

本地时区由 `/etc/localtime` 控制

时区文件位于 `/usr/share/zoneinfo`

* 里面包含了时区和时区的别名

如果想要手动修改时区

* 要么将 `/usr/share/zoneinfo` 中的一个文件拷贝到 `/etc/localtime`
* 或者使用 Linux 上的时区工具

### 7.5.2 Network Time

_Network Time Protocol (NTP)_ 使用远程服务器维护时间

1. 找到距离 ISP 最近的 NTP 时间服务器
2. 将服务器配置在 `/etc/ntpd.conf` 中
3. 在启动时运行 `dtpdate server`
4. 之后运行 `ntpd`

## 7.6 Scheduling Recurring Tasks with cron

Unix cron 服务会以固定频率重复运行程序

由 cron 运行的程序被称为 _cron job_

通常，可以运行 `crontab` 命令，在 crontab 文件中加入条目

```
15 09 * * * /home/juser/bin/spmake
```

前五个参数分别为：

* Minute
* Hour
* Day of month
* Month
* Day of week

其中，`*` 代表所有的值，任意一个域可以使用 `,` 分隔多个值

可以将 cron job 编辑在文件中，通过 `cron file` 命令安装 cron 文件

也可以通过 `cron -e` 编辑、安装一步到位

### 7.6.3 The Future of cron

cron 工具是 Linux 最古老的组件之一

## 7.7 Scheduling One-Time Tasks with at

使用 `at` 服务运行将来某个时刻只执行一次的命令

```bash
$ at 22:30
at> myjob
```

使用 `CTRL-D` 结束输入 (`at` 工具使用标准输入读取命令)

`atq` 用于确认

`atrm` 用于删除

## 7.8 Understanding User IDs and User Switching

有两种方法修改 user ID：

* setuid executable
* 通过 `setuid()` 家族的系统调用

内核有一些基本的规则，限制了一个进程能做什么或不能做什么：

* 运行于 root (UID == 0) 的进程可以使用 `setuid()` 变成任何用户的进程
* 不运行于 root 的进程对于 `setuid()` 的使用具有很严格的限制，大部分状况下不行
* 任何进程可以执行 setuid 程序，只要具有足够的文件权限

### 7.8.1 Process Ownership, Effective UID, Real UID, and Saved UID

每个进程拥有多个 UID

* effective UID (euid) - 定义了进程的访问权限
* real UID (ruid) - 标识了谁启动了该进程

当使用 `setuid()` 时，内核修改了 euid，但保留了 ruid

创建该进程的用户依然可以发送信号给该进程，或杀死进程，即使进程由其他用户执行

Linux 默认 euid 和 ruid 相同

`ps` 等程序默认展示 euid

```bash
$ ps -eo pid,euser,ruser,comm
```

此外，还有 _saved user ID_ 和 _file system user ID_

利用运行在 root 用户的程序漏洞是系统入侵的主要方法

## 7.9 User Identification and Authentication

* Identification - who users are
* Authentication - prove that they are who they say they are
* Authorization - define and limit what users are allowed to do

所有的 authentication 发生在用户空间中，因为内核不知道 username

1. 进程通过 `geteuid()` 系统调用向内核询问自己的 euid
2. 进程打开 `/etc/passwd` 并读取
3. 每次读取一行
4. 将每一行进行分割，找到 user ID
5. 将 user ID 与内核返回的 ID 进行比较，若相同，则找到了对应的 username 并进行使用

### 7.9.1 Using Libraries for User Information

在通过 `geteuid()` 系统调用获得 euid 后

使用标准库中的 `getpwuid()` 获得 username

关于验证密码，在传统的实现方式中的局限：

* 对于加密协议有没系统性的标准
* 假设能够访问加密后的密码

## 7.10 PAM

为了使认证更具灵活性

1995 年，Sun Microsystems 提出了新标准：

_Pluggable Authentication Modules (PAM)_

* 一系列用于认证的共享库

由于支持多种认证场景

PAM 部署了大量动态加载的认证模块

每个模块执行特定的功能

PAM 几乎支持了 Linux 系统上需要认证的每一个程序

### 7.10.1 PAM Configuration

PAM 的配置文件位于 `/etc/pam.d` 目录下

配置文件中的每一行包含：

* Function Type - 用户程序想要 PAM 执行的功能
  * auth - 认证用户
  * account - 检查用户账号状态 (是否被授权做某事)
  * session - 仅在当前会话中执行 sth.
  * password - 修改用户的密码或其它证书
* Control Argument - PAM 在当前行执行成功或失败后执行的动作
  * sufficient
    * 如果当前行认证成功，则 PAM 不需要再检查其它行
    * 否则 PAM 需要继续检查其它行
  * requisite
    * 如果当前行认证成功，PAM 还需要检查其它行
    * 如果当前行认证失败，则总体认证失败
  * required
    * 如果当前行认证成功，PAM 还需要检查其它行
    * 如果当前行认证失败，PAM 继续检查其它行，但总会返回不成功的认证
* Module - 该行需要使用的模块
  * 可以在模块名后面带参数

Function Type 和 Module 共同决定了 PAM 的动作

### 7.10.2 Notes on PAM

### 7.10.3 PAM and Passwords

`/etc/login.defs` 中保存了 shadow 文件中使用的加密算法

但很少在安装了 PAM 的机器上使用

因为 PAM 的配置文件中包含了这些信息

PAM 从哪里获得密码加密的模式呢？

```bash
$ grep password.*unix /etc/pam.d/*
/etc/pam.d/password-auth:password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
/etc/pam.d/password-auth-ac:password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
/etc/pam.d/system-auth:password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
/etc/pam.d/system-auth-ac:password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
```

可以得知，PAM 使用 SHA-512 来加密密码

但加密只用于设置密码时 (password)，而不是 PAM 认证密码时 (auth)

* 对于 auth 功能，没有任何的加密参数
* `pam_unix.so` 试图将算法猜出来，直到没有可以尝试的算法

---

