# 第一部分 Linux 命令行

## 第1章 初识Linuxshell

### 1.1 什么是 Linux

内核创建了第一个进程(称为init进程)来启动系统上所有其他进程。当内核启动时，它会将init进程加载到虚拟内存中。内核在启动任何其他进程时，都会在虚拟内存中给新进程分配一块专有区域来存储该进程用到的数据和代码。

一些Linux发行版使用一个表来管理在系统开机时要自动启动的进程。在Linux系统上，这个表通常位于专门文件/etc/inittab中。

另外一些系统(比如现在流行的Ubuntu Linux发行版)则采用/etc/init.d目录，将开机时启动或停止某个应用的脚本放在这个目录下。这些脚本通过/etc/rcX.d目录下的入口(entry)1启动，这里的X代表运行级(run level)。

```
[admin@host-11-20-246-130 init.d]$ pwd
/etc/init.d
[admin@host-11-20-246-130 init.d]$ ll
总用量 28
-rw-r--r-- 1 root root 13948 9月  16 2015 functions
-rwxr-xr-x 1 root root  3852 8月   5 2016 ifritd
-rwxr-xr-x 1 root root  1270 7月  17 2015 nginx
-rw-r--r-- 1 root root  1160 11月 20 2015 README
```

```
[admin@host-11-20-246-130 etc]$ cd /etc/rc
rc0.d/    rc1.d/    rc2.d/    rc3.d/    rc4.d/    rc5.d/    rc6.d/    rc.d/     rc.local
[admin@host-11-20-246-130 etc]$ cd /etc/rc0.d/
[admin@host-11-20-246-130 rc0.d]$ ll
总用量 0
lrwxrwxrwx 1 root root 16 8月   5 2016 K50ifritd -> ../init.d/ifritd
```

第 3 章 基本的 bash shell 命令

一些帮助提示

```
man man

info vim

help

空格翻页 q 退出
```