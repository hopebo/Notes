# Bash Commands

## Execute commands on the remote server using `ssh`

```bash
ssh username@hostname "cd ~; pwd"
```

Execute commands on the remote server. If the commands need some interaction,
we need system to allocate tty for us using `-t` option.

```bash
ssh username@hostname 'bash -s' < test.sh helloworld
```

In this way we can execute the contents in local script with options. But if we
need interaction, we can only use the following way:

```bash
ssh -t username@hostname "`cat test.sh`"

ssh username@hostname test.sh helloworld
```

Execute remote script file in remote server.


## Remote synchronization using `rsync`

```bash
# Copy the whole folder including hidden files.
rsync -avz ~/.emacs.d/ username@hostname:~/.emacs.d

# Ignore hidden files directly in the folder, but will include hidden files in subdirectory.
rsync -avz ~/.emacs.d/* username@hostname:~/.emacs.d
```

`-exclude=dir` and `-exclude-from=file.list` can help us to exclude some files.

## Expect Manual

[Operators](https://www.tcl.tk/man/tcl/TclCmd/expr.htm)

[Tcl Commands](http://tcl.tk/man/tcl8.5/TclCmd/contents.htm)

## 把 home 目录软链接到其他目录
```bash
-sudo mkdir /workspace/username
# 将现有文件搬运到修改目录下
-sudo cp -r /home/username/ /workspace/username
-sudo rm -r /home/username
-sudo ln -s /workspace/username /home/username

# 修改链接文件的权限，需要加上选项-h
chown -h
```
加上 -s 为符号链接，也称作软链接，软链接可以对目录或者文件；而硬链接只能对文件。软链接会生成新的链接文件，里面存放的是被链接文件或目录的地址。硬链接不会生成新的文件，只是会把被链接文件在多个目录中显示。

## 查看所有进程
```bash
ps aux | less
```

## 搜索内容
```bash
grep -i
```
忽略大小写

# 系统日志

## 权限认证登陆日志
Red Hat family distributions (including CentOS and Fedora) use `/var/log/messages` and `/var/log/secure` where Debian-family distributions (Ubuntu) use `/var/log/syslog` and `/var/log/auth.log`

```bash
# Ubuntu
less /var/log/auth.log

# Centos
less /var/log/secure

```

PAM(Pluggable Authentication Module) 模块是用来认证用户权限的，所有用户的登陆、登出、以及被拒绝的记录。部分输出如下所示。

```
Jul 14 15:03:58 hostname sshd[16409]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=30.211.64.195  user=hope.lb
Jul 14 15:03:59 hostname su[16468]: (to root) root on none
Jul 14 15:03:59 hostname su[16468]: pam_unix(su:session): session opened for user root by (uid=0)
Jul 14 15:03:59 hostname su[16468]: pam_unix(su:session): session closed for user root
Jul 14 15:04:00 hostname sshd[16409]: Failed password for hope.lb from 30.211.64.195 port 50525 ssh2
Jul 14 15:04:04 hostname sshd[16409]: Accepted password for hope.lb from 30.211.64.195 port 50525 ssh2
Jul 14 15:04:04 hostname systemd-logind[3741]: New session 415035 of user hope.lb.
Jul 14 15:04:04 hostname sshd[16409]: pam_unix(sshd:session): session opened for user hope.lb by (uid=0)
```

## 登陆记录
- `/var/log/btmp`: recordings of failed login attempts
- `/var/run/utmp`: current login state, by user
- `/var/log/wtmp`: login/logout history

```
last 
或者
last -f /var/log/wtmp
```
命令来查看登陆/登出历史。

```
w
```
命令也可以查看当前活跃用户和执行的命令。

## 命令日志
每个用户执行的命令都保存在 home 目录下的`.bash_history`下，可以直接查看。
```
history
```
命令可查看当前用户执行过的命令。

## Linux 启动过程和硬件故障

In Linux and Unix-like computers, booting and startup are two distinct phases of the sequence of events that take place when the computer is powered on.

The boot processes (*BIOS* or *UEFI*, *MBR*, and *GRUB*) take the initialization of the system to the point where the kernel is loaded into memory and connected to the initial ramdisk (*initrd* or *initramfs*), and *systemd* is started.

The startup processes then pick up the baton and complete the initialization of the operating system. In the very early stages of initialization, logging daemons such as *syslogd* or *rsyslogd* are not yet up and running. To avoid losing notable error messages and warnings from this phase of initialization, the kernel contains a *ring buffer* that it uses as a message store.

A ring buffer is a memory space reserved for messages. It is simple in design, and of a fixed size. When it is full, newer messages overwrite the oldest messages. Conceptually it can be thought of as a “*circular buffer*.”

The kernel ring buffer stores information such as the initialization messages of device drivers, messages from hardware, and messages from kernel modules. Because it contains these low-level startup messages, the ring buffer is a good place to start an investigation into hardware errors or other startup issues.

```bash
dmesg -H
```
命令可以自动将时间戳转换为时间，并自动通过`less`命令输出。

可以看到一些示例输出：
```
[Dec15 10:22] Linux version 5.10.60-002.os7.x86_64 (admin@1f5aaeddc43d) (gcc (GCC) 10.2.1, GNU ld version 2.35-12.2.os7) #1 SMP Wed Sep 29 23:47:32 CST 2021
[  +0.000000] Command line: BOOT_IMAGE=/vmlinuz-5.10.60-002.5000.os7.x86_64 root=UUID=1cf7d3969d4 ro console=ttyS0,115200 LANG=en_US.UTF-8 fsck.repair=yes virtio_ring.vring_force_dma_api=1 vring_force_dma_ap
[  +0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
[  +0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
[  +0.000000] x86/fpu: Supporting XSAVE feature 0x040: 'AVX-512 Hi256'
[  +0.000000] x86/fpu: xstate_offset[7]: 1408, xstate_sizes[7]: 1024 (default)
[  +0.000000] x86/fpu: Enabled xstate features 0xe7, context size is 2432 bytes, using 'compacted' format.
[  +0.000000] signal: max sigframe size: 3632
[  +0.000000] BIOS-provided physical RAM map:
[  +0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
[  +0.000000] BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
```

`dmesg`支持按照日志级别打印信息：
```
# 种类包括 emerg、alert、crit、err、warn、notice、info、debug。比如 Out of memory 都属于 err 级别。
dmesg -l err

[Dec15 10:22] integrity: Unable to open file: /etc/keys/x509_ima.der (-2)
[  +0.000002] integrity: Unable to open file: /etc/keys/x509_evm.der (-2)
[  +0.072551] ip_local_port_range: prefer different parity for start/end values.
[Dec15 10:23] SELinux:  Runtime disable is deprecated, use selinux=0 on the kernel cmdline.
[  +0.623192] nouveau 0000:00:08.0: unknown chipset (b72000a1)
[  +0.018649] nouveau 0000:00:09.0: unknown chipset (b72000a1)
[  +0.015792] nouveau 0000:00:0a.0: unknown chipset (b72000a1)
[  +0.013732] nouveau 0000:00:0b.0: unknown chipset (b72000a1)
[Feb16 15:36] EXT4-fs (vda2): Project quota feature not enabled. Cannot enable project quota enforcement.
[Apr25 23:15] Out of memory: Killed process 38635 (gdb) total-vm:745149304kB, anon-rss:735165040kB, file-rss:4096kB, shmem-rss:0kB, UID:1350831 pgtables:1440924kB oom_score_adj:0
[Jul12 10:15] Out of memory: Killed process 74278 (mysqld) total-vm:1163710880kB, anon-rss:737936532kB, file-rss:0kB, shmem-rss:0kB, UID:1376958 pgtables:1446324kB oom_score_adj:0
[Jul12 10:38] Out of memory: Killed process 107170 (mysqld) total-vm:1163723168kB, anon-rss:736785036kB, file-rss:0kB, shmem-rss:0kB, UID:1376958 pgtables:1444052kB oom_score_adj:0
[Jul12 11:05] Out of memory: Killed process 12811 (mysqld) total-vm:1163710884kB, anon-rss:735752424kB, file-rss:0kB, shmem-rss:0kB, UID:1376958 pgtables:1442040kB oom_score_adj:0
[Jul12 11:38] Out of memory: Killed process 69357 (mysqld) total-vm:1163723172kB, anon-rss:732997508kB, file-rss:0kB, shmem-rss:0kB, UID:1376958 pgtables:1436636kB oom_score_adj:0
[Jul12 12:04] Out of memory: Killed process 104194 (mysqld) total-vm:1163731932kB, anon-rss:730923120kB, file-rss:0kB, shmem-rss:0kB, UID:1376958 pgtables:1432596kB oom_score_adj:0
```

`dmesg`支持按照日志功能种类打印：
```
# 种类包括 kern、user、mail、daemon、auth、syslog、lpr、news。
dmesg -f daemon
```

[How to Use the dmesg Command on Linux](https://www.howtogeek.com/449335/how-to-use-the-dmesg-command-on-linux/)

## 系统通用日志
```
less /var/log/messages
```
通用日志，记录了所有全局系统相关的活动，包括系统启动过程中的、mail、cron、daemon、kern、auth 的。

### 内存溢出
```
cat /var/log/messages | grep -i "Out of memory"
或者
cat /var/log/messages | grep -i "oom-killer"
```