# Chapter 10 - Network Applications and Services

Created by : Mr Dk.

2019 / 07 / 09 23:58

@NUAA, Nanjing, Jiangsu, China

---

又是水得不行的一章... 🤨

## 10.2 Network Servers

- *httpd*, *apache*, *apache2*：Web servers
- *sshd*：Secure shell
- *postfix*, *qmail*, *sendmail*：Mail servers
- *cupsd*：Print server
- *nfsd*, *mountd*：Network filesystem daemons
- *smbd*, *nmbd*：WIndows file-sharing daemons
- *rpcbind*：Remote procedure call (RPC) portmap service deamon

网络服务器的共同特征：多进程。

- 至少一个进程监听端口
- 当接收到连接后，监听进程使用 `fork()` 创建新的子进程处理连接
- 同时，监听进程继续监听端口

其中，调用 `fork()` 给系统带来了较大开销。因此，一些高性能的 TCP 服务器会在启动时创建大量工作进程：worker process。

## 10.3 Secure Shell (SSH)

支持远程登录、程序执行、文件共享。使用公钥密码体制替代了老旧的、不安全的 `telnet` 和 `rlogin`。*OpenSSH* 是最受欢迎的免费 SSH 实现。

### 10.3.1 The SSHD Server

运行 `sshd` 服务需要一个配置文件和密钥：`/etc/ssh`

- 服务器配置文件：`sshd_config`
- 客户端配置文件：`ssh_config`

服务器配置文件中可以配置监听端口和密钥路径。

#### Host Keys

SSH version 1 只有 RSA 密钥，SSH version 2 有 RSA 和 DSA 密钥。使用 `ssh-keygen` 可以生成公私钥对。

### 10.3.2 The SSH Client

```bash
ssh username@host
```

在 `.ssh/known_hosts` 中保存了远程服务器的公钥。如果远程服务器的公钥与以前的公钥发生了冲突，比如替换了硬件或操作系统，将会产生警告。将以前的公钥记录删除即可。

#### SSH File Transfer Clients

```bash
scp user1@host1:file user2@host2:dir
```

`sftp` 使用 `get` 和 `put` 命令，服务器需要运行 `sftp-server`。

## 10.4 The inetd and xinetd Daemons

为了避免分别运行每种服务的麻烦，`inetd` 读取配置文件，监听对应的端口。当接受到连接时，`inetd` 将连接分配到对应的进程。

## 10.5 Diagnostic Tools

### 10.5.1 lsof

`lsof` 可用于追踪所有打开的文件，也可以查看所有正在监听端口的程序：

```console
$ lsof -i
COMMAND     PID            USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
systemd-r   617 systemd-resolve   12u  IPv4   14501      0t0  UDP localhost:domain
systemd-r   617 systemd-resolve   13u  IPv4   14502      0t0  TCP localhost:domain (LISTEN)
v2ray       734            root    3u  IPv6   17736      0t0  TCP *:23591 (LISTEN)
sshd        770            root    3u  IPv4   17201      0t0  TCP *:ssh (LISTEN)
sshd        770            root    4u  IPv6   17322      0t0  TCP *:ssh (LISTEN)
ssserver   1178            root    4u  IPv4   20298      0t0  TCP client-23-254-225-164.hostwindsdns.com:12306 (LISTEN)
ssserver   1178            root    5u  IPv4   20299      0t0  UDP client-23-254-225-164.hostwindsdns.com:12306
ssserver   1178            root    8u  IPv4   20300      0t0  UDP *:50810
sshd      17879            root    3u  IPv4 1012568      0t0  TCP client-23-254-225-164.hostwindsdns.com:ssh->157.0.78.120:65096 (ESTABLISHED)
```

使用 `-n` 选项禁止 IP 地址和熟知端口的反向解析：

```console
$ lsof -n -i
COMMAND     PID            USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
systemd-r   617 systemd-resolve   12u  IPv4   14501      0t0  UDP 127.0.0.53:domain
systemd-r   617 systemd-resolve   13u  IPv4   14502      0t0  TCP 127.0.0.53:domain (LISTEN)
v2ray       734            root    3u  IPv6   17736      0t0  TCP *:23591 (LISTEN)
sshd        770            root    3u  IPv4   17201      0t0  TCP *:ssh (LISTEN)
sshd        770            root    4u  IPv6   17322      0t0  TCP *:ssh (LISTEN)
ssserver   1178            root    4u  IPv4   20298      0t0  TCP 23.254.225.164:12306 (LISTEN)
ssserver   1178            root    5u  IPv4   20299      0t0  UDP 23.254.225.164:12306
ssserver   1178            root    8u  IPv4   20300      0t0  UDP *:50810
sshd      17879            root    3u  IPv4 1012568      0t0  TCP 23.254.225.164:ssh->157.0.78.120:65096 (ESTABLISHED)
```

并可以使用各种过滤规则。

### 10.5.2 tcpdump

### 10.5.3 netcat

```bash
netcat host port
```

监听一个特定端口：

```bash
netcat -l -p port_number
```

### 10.5.4 Port Scanning

Network Mapper (Nmap) 程序可以扫描一台机器所有开放的端口：

```console
$ nmap 103.235.46.39

Starting Nmap 7.60 ( https://nmap.org ) at 2019-07-09 13:32 UTC
Nmap scan report for 103.235.46.39
Host is up (0.20s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 20.10 seconds
```

## 10.6 Remote Procedure Call (RPC)

在远程机器上调用函数，并返回一个结果码或消息。RPC 的实现需要将程序编号和传输层端口号映射。

## 10.9 Sockets: How Processes Communicate with the Network

Socket 是进程通过内核访问网络的接口，表示了用户空间和内核之间的界限，也被用于进程间通信 (inter-process communication, IPC)。有多种不同类型的 socket：

- stream socket：`SOCK_STREAM` 用于 TCP
- datagram sockets：`SOCK_DGRAM` 用于 UDP

## 10.10 Unix Domain Sockets

在同一台机器上运行的进程，使用进程间通信进行沟通。进程使用本地 IP 地址 *localhost* 进行通信，使用的是一种特殊类型的 socket：Unix domain socket。Unix domain socket 实际上没有使用网络，甚至都不需要网络被配置就能使用，也不需要绑定任何的 socket 文件。进程创建的未命名 socket 可以直接通过地址被其它进程共享。

### 10.10.1 Advantages for Developers

- 便于访问控制
- 不需要进入协议栈，性能更好

### 10.10.2 Listing Unix Domain Sockets

查看所有的 Unix domain sockets：

```console
$ lsof -U
COMMAND     PID             USER   FD   TYPE             DEVICE SIZE/OFF    NODE NAME
systemd       1             root   14u  unix 0xffff965bb6e90c00      0t0   11703 /run/systemd/notify type=DGRAM
systemd       1             root   15u  unix 0xffff965bb6e91800      0t0   11704 type=DGRAM
systemd       1             root   16u  unix 0xffff965bb6e90000      0t0   11705 type=DGRAM
systemd       1             root   17u  unix 0xffff965bb9bd1000      0t0   11706 /run/systemd/private type=STREAM
systemd       1             root   30u  unix 0xffff965bb981f000      0t0   11719 /run/udev/control type=SEQPACKET
systemd       1             root   32u  unix 0xffff965bb981fc00      0t0   11721 /run/lvm/lvmpolld.socket type=STREAM
systemd       1             root   33u  unix 0xffff965bb981e800      0t0   11723 /run/lvm/lvmetad.socket type=STREAM
systemd       1             root   34u  unix 0xffff965bb981e000      0t0   11725 /run/systemd/journal/stdout type=STREAM
systemd       1             root   35u  unix 0xffff965bb981f800      0t0   11727 /run/systemd/journal/socket type=DGRAM
systemd       1             root   36u  unix 0xffff965bbcb19400      0t0   12144 /run/systemd/journal/dev-log type=DGRAM
systemd       1             root   37u  unix 0xffff965bb6865000      0t0   12147 /run/systemd/journal/syslog type=DGRAM
systemd       1             root   49u  unix 0xffff965bbac4b800      0t0   12774 type=DGRAM
systemd       1             root   51u  unix 0xffff965bb684f000      0t0   12843 /run/systemd/journal/stdout type=STREAM
systemd       1             root   52u  unix 0xffff965bbcb4e000      0t0   12844 /run/systemd/journal/stdout type=STREAM
systemd       1             root   54u  unix 0xffff965bbc927400      0t0   14223 /run/systemd/journal/stdout type=STREAM
systemd       1             root   55u  unix 0xffff965bb6877c00      0t0   14380 /run/systemd/journal/stdout type=STREAM
systemd       1             root   56u  unix 0xffff965bb684e000      0t0   13459 type=DGRAM
systemd       1             root   57u  unix 0xffff965bbca9a400      0t0   15030 /run/uuidd/request type=STREAM
systemd       1             root   58u  unix 0xffff965bb684e400      0t0   13460 type=DGRAM
systemd       1             root   59u  unix 0xffff965bbcfc9400      0t0   15036 @ISCSIADM_ABSTRACT_NAMESPACE type=STREAM
systemd       1             root   60u  unix 0xffff965bb65c5c00      0t0   13531 /run/systemd/journal/stdout type=STREAM
systemd       1             root   61u  unix 0xffff965bbab35800      0t0   15046 /run/acpid.socket type=STREAM
systemd       1             root   62u  unix 0xffff965bbab35c00      0t0   15058 /var/run/dbus/system_bus_socket type=STREAM
systemd       1             root   63u  unix 0xffff965bbab35400      0t0   15069 /run/snapd.socket type=STREAM
systemd       1             root   64u  unix 0xffff965bbca9b400      0t0   15071 /run/snapd-snap.socket type=STREAM
systemd       1             root   66u  unix 0xffff965bbaba1c00      0t0   15082 /var/lib/lxd/unix.socket type=STREAM
systemd       1             root   68u  unix 0xffff965bbc9a1000      0t0   15359 type=STREAM
systemd       1             root   73u  unix 0xffff965bbabe3400      0t0   15360 /run/systemd/journal/stdout type=STREAM
systemd       1             root   74u  unix 0xffff965bbcb12c00      0t0   15470 /run/systemd/journal/stdout type=STREAM
systemd       1             root   75u  unix 0xffff965bbce4e000      0t0   15549 /run/systemd/journal/stdout type=STREAM
systemd       1             root   76u  unix 0xffff965bbce4e800      0t0   17309 /run/systemd/journal/stdout type=STREAM
systemd       1             root   78u  unix 0xffff965bbcc37800      0t0   15799 /run/systemd/journal/stdout type=STREAM
systemd       1             root   79u  unix 0xffff965bbaf47c00      0t0   17077 /run/systemd/journal/stdout type=STREAM
systemd       1             root   80u  unix 0xffff965bbcd91000      0t0   18812 /run/systemd/journal/stdout type=STREAM
systemd       1             root   81u  unix 0xffff965bbcd43400      0t0   16623 /run/systemd/journal/stdout type=STREAM
systemd       1             root   82u  unix 0xffff965bbcdff000      0t0   16628 /run/systemd/journal/stdout type=STREAM
systemd       1             root   84u  unix 0xffff965bbccca000      0t0   16630 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root    3u  unix 0xffff965bb981e000      0t0   11725 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root    4u  unix 0xffff965bb981f800      0t0   11727 /run/systemd/journal/socket type=DGRAM
systemd-j   410             root    5u  unix 0xffff965bbcb19400      0t0   12144 /run/systemd/journal/dev-log type=DGRAM
systemd-j   410             root   14u  unix 0xffff965bbcac2800      0t0   12522 type=DGRAM
systemd-j   410             root   16u  unix 0xffff965bbc927400      0t0   14223 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   17u  unix 0xffff965bb6877c00      0t0   14380 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   18u  unix 0xffff965bb684f000      0t0   12843 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   19u  unix 0xffff965bbcb4e000      0t0   12844 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   20u  unix 0xffff965bb65c5c00      0t0   13531 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   21u  unix 0xffff965bbabe3400      0t0   15360 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   22u  unix 0xffff965bbcb12c00      0t0   15470 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   23u  unix 0xffff965bbce4e000      0t0   15549 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   25u  unix 0xffff965bbce4e800      0t0   17309 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   27u  unix 0xffff965bbcc37800      0t0   15799 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   28u  unix 0xffff965bbaf47c00      0t0   17077 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   29u  unix 0xffff965bbcd91000      0t0   18812 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   30u  unix 0xffff965bbcd43400      0t0   16623 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   31u  unix 0xffff965bbcdff000      0t0   16628 /run/systemd/journal/stdout type=STREAM
systemd-j   410             root   33u  unix 0xffff965bbccca000      0t0   16630 /run/systemd/journal/stdout type=STREAM
systemd-u   420             root    1u  unix 0xffff965bb684f800      0t0   12404 type=STREAM
systemd-u   420             root    2u  unix 0xffff965bb684f800      0t0   12404 type=STREAM
systemd-u   420             root    3u  unix 0xffff965bb981f000      0t0   11719 /run/udev/control type=SEQPACKET
systemd-u   420             root    5u  unix 0xffff965bbc927c00      0t0   12463 type=DGRAM
systemd-u   420             root    7u  unix 0xffff965bbc9a1c00      0t0   12543 type=DGRAM
systemd-u   420             root    8u  unix 0xffff965bbcac2400      0t0   12544 type=DGRAM
lvmetad     421             root    1u  unix 0xffff965bbcb4e400      0t0   12460 type=STREAM
lvmetad     421             root    2u  unix 0xffff965bbcb4e400      0t0   12460 type=STREAM
lvmetad     421             root    3u  unix 0xffff965bb981e800      0t0   11723 /run/lvm/lvmetad.socket type=STREAM
systemd-t   519 systemd-timesync    1u  unix 0xffff965bb65c5800      0t0   13530 type=STREAM
systemd-t   519 systemd-timesync    2u  unix 0xffff965bb65c5800      0t0   13530 type=STREAM
systemd-t   519 systemd-timesync    3u  unix 0xffff965bbab47400      0t0   13604 type=DGRAM
systemd-t   519 systemd-timesync    7u  unix 0xffff965bbcb12000      0t0   13607 type=DGRAM
systemd-t   519 systemd-timesync    8u  unix 0xffff965bbcb12800      0t0   13608 type=DGRAM
systemd-t   519 systemd-timesync    9u  unix 0xffff965bbcb13000      0t0   13609 type=DGRAM
systemd-t   519 systemd-timesync   10u  unix 0xffff965bbcb13800      0t0   13610 type=DGRAM
systemd-n   599  systemd-network    1u  unix 0xffff965bbc927000      0t0   14222 type=STREAM
systemd-n   599  systemd-network    2u  unix 0xffff965bbc927000      0t0   14222 type=STREAM
systemd-n   599  systemd-network    4u  unix 0xffff965bbcb19800      0t0   14265 type=DGRAM
systemd-n   599  systemd-network   11u  unix 0xffff965bbab47800      0t0   14272 type=DGRAM
systemd-n   599  systemd-network   12u  unix 0xffff965bbcb18000      0t0   14273 type=DGRAM
systemd-n   599  systemd-network   13u  unix 0xffff965bbdb10800      0t0   14274 type=DGRAM
systemd-n   599  systemd-network   14u  unix 0xffff965bbcb4f000      0t0   14275 type=DGRAM
systemd-n   599  systemd-network   19u  unix 0xffff965bbcfc8800      0t0   15061 type=STREAM
systemd-r   617  systemd-resolve    1u  unix 0xffff965bb6876400      0t0   14379 type=STREAM
systemd-r   617  systemd-resolve    2u  unix 0xffff965bb6876400      0t0   14379 type=STREAM
systemd-r   617  systemd-resolve    3u  unix 0xffff965bb684fc00      0t0   14481 type=DGRAM
systemd-r   617  systemd-resolve   14u  unix 0xffff965bbca9a000      0t0   15060 type=STREAM
acpid       725             root    0u  unix 0xffff965bbab35800      0t0   15046 /run/acpid.socket type=STREAM
acpid       725             root    3u  unix 0xffff965bbabe2000      0t0   15285 type=DGRAM
dbus-daem   726       messagebus    1u  unix 0xffff965bbabe2400      0t0   15358 type=STREAM
dbus-daem   726       messagebus    2u  unix 0xffff965bbabe2400      0t0   15358 type=STREAM
dbus-daem   726       messagebus    3u  unix 0xffff965bbab35c00      0t0   15058 /var/run/dbus/system_bus_socket type=STREAM
dbus-daem   726       messagebus    6u  unix 0xffff965bb684ec00      0t0   15382 type=DGRAM
dbus-daem   726       messagebus    8u  unix 0xffff965bb6876c00      0t0   15389 type=STREAM
dbus-daem   726       messagebus    9u  unix 0xffff965bb6877400      0t0   15390 type=STREAM
dbus-daem   726       messagebus   10u  unix 0xffff965bbcfc9000      0t0   15391 /var/run/dbus/system_bus_socket type=STREAM
dbus-daem   726       messagebus   11u  unix 0xffff965bbcfc8000      0t0   15392 /var/run/dbus/system_bus_socket type=STREAM
dbus-daem   726       messagebus   12u  unix 0xffff965bbca6f400      0t0   15393 /var/run/dbus/system_bus_socket type=STREAM
dbus-daem   726       messagebus   13u  unix 0xffff965bba8a2c00      0t0   16957 /var/run/dbus/system_bus_socket type=STREAM
dbus-daem   726       messagebus   14u  unix 0xffff965bbae2dc00      0t0   17194 /var/run/dbus/system_bus_socket type=STREAM
dbus-daem   726       messagebus   15u  unix 0xffff965bbafa8800      0t0   17213 /var/run/dbus/system_bus_socket type=STREAM
dbus-daem   726       messagebus   16u  unix 0xffff965bba8a2800      0t0   17605 /var/run/dbus/system_bus_socket type=STREAM
dbus-daem   726       messagebus   17u  unix 0xffff965bba93f800      0t0   17707 /var/run/dbus/system_bus_socket type=STREAM
lxcfs       730             root    1u  unix 0xffff965bbcb13c00      0t0   15468 type=STREAM
lxcfs       730             root    2u  unix 0xffff965bbcb13c00      0t0   15468 type=STREAM
systemd-l   731             root    1u  unix 0xffff965bbcb12400      0t0   15548 type=STREAM
systemd-l   731             root    2u  unix 0xffff965bbcb12400      0t0   15548 type=STREAM
systemd-l   731             root    3u  unix 0xffff965bbb805000      0t0   17183 type=DGRAM
systemd-l   731             root   12u  unix 0xffff965bbae2c400      0t0   17193 type=STREAM
v2ray       734             root    1u  unix 0xffff965bbe38b400      0t0   15798 type=STREAM
v2ray       734             root    2u  unix 0xffff965bbe38b400      0t0   15798 type=STREAM
cron        739             root    1u  unix 0xffff965bbcd42400      0t0   16046 type=STREAM
cron        739             root    2u  unix 0xffff965bbcd42400      0t0   16046 type=STREAM
rsyslogd    740           syslog    3u  unix 0xffff965bb6865000      0t0   12147 /run/systemd/journal/syslog type=DGRAM
rsyslogd    740           syslog    6u  unix 0xffff965bbaf17000      0t0   16934 type=DGRAM
networkd-   741             root    1u  unix 0xffff965bbcdff400      0t0   16211 type=STREAM
networkd-   741             root    2u  unix 0xffff965bbcdff400      0t0   16211 type=STREAM
networkd-   741             root    3u  unix 0xffff965bba8a3400      0t0   17604 type=STREAM
accounts-   743             root    1u  unix 0xffff965bbccca800      0t0   16368 type=STREAM
accounts-   743             root    2u  unix 0xffff965bbccca800      0t0   16368 type=STREAM
accounts-   743             root    5u  unix 0xffff965bba8a3c00      0t0   16956 type=STREAM
sshd        770             root    1u  unix 0xffff965bbaef5c00      0t0   17076 type=STREAM
sshd        770             root    2u  unix 0xffff965bbaef5c00      0t0   17076 type=STREAM
polkitd     780             root    6u  unix 0xffff965bbae2d400      0t0   17212 type=STREAM
unattende   791             root    1u  unix 0xffff965bbce4f800      0t0   17307 type=STREAM
unattende   791             root    2u  unix 0xffff965bbce4f800      0t0   17307 type=STREAM
unattende   791             root    4u  unix 0xffff965bba8e8400      0t0   17706 type=STREAM
systemd     967             root    1u  unix 0xffff965bbcd90000      0t0   18804 type=STREAM
systemd     967             root    2u  unix 0xffff965bbcd90000      0t0   18804 type=STREAM
systemd     967             root    3u  unix 0xffff965bbaf3c800      0t0   18842 type=DGRAM
systemd     967             root   14u  unix 0xffff965bbcd90c00      0t0   18880 /run/user/0/systemd/notify type=DGRAM
systemd     967             root   15u  unix 0xffff965bbcd90800      0t0   18881 type=DGRAM
systemd     967             root   16u  unix 0xffff965bbaf3cc00      0t0   18882 type=DGRAM
systemd     967             root   17u  unix 0xffff965bbaf3d800      0t0   18883 /run/user/0/systemd/private type=STREAM
systemd     967             root   22u  unix 0xffff965bbaeb1800      0t0   18887 /run/user/0/gnupg/S.gpg-agent.extra type=STREAM
systemd     967             root   23u  unix 0xffff965bbcd43800      0t0   18888 /run/user/0/gnupg/S.gpg-agent type=STREAM
systemd     967             root   24u  unix 0xffff965bbcd43000      0t0   18889 /run/user/0/gnupg/S.gpg-agent.ssh type=STREAM
systemd     967             root   25u  unix 0xffff965bbcd42c00      0t0   18890 /run/user/0/gnupg/S.gpg-agent.browser type=STREAM
systemd     967             root   26u  unix 0xffff965bbcd43c00      0t0   18891 /run/user/0/gnupg/S.dirmngr type=STREAM
(sd-pam     974             root    1u  unix 0xffff965bbcd90000      0t0   18804 type=STREAM
(sd-pam     974             root    2u  unix 0xffff965bbcd90000      0t0   18804 type=STREAM
(sd-pam     974             root    7u  unix 0xffff965bbaf3c000      0t0   18820 type=DGRAM
sshd      18762             root    4u  unix 0xffff965bba97c800      0t0 1019393 type=DGRAM
```

