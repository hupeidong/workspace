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

## 第 3 章 基本的 bash shell 命令

### 1 bash 手册

一些帮助提示

```
man man

info vim

help

空格翻页 q 退出
```

### 2 浏览文件系统

**窍门** 你将会发现Linux使用正斜线(/)而不是反斜线(\)在文件路径中划分目录。在Linux中，反斜线用来标识转义字符，要是用在文件路径中的话会导致各种各样的问题。如果你之前用的是Windows环境，就需要一点时间来适应。

还可以从Linux虚拟目录中的任何一级跳回主目录:

```
christine@server01:/var/log$ cd
christine@server01:~$
christine@server01:~$ pwd /home/christine
christine@server01:~$
```

### 3 文件和目录列表

`ls -d` （只显示当前文件夹）

可用带-F参数的ls命令轻松区分文件和目录。使用-F参数可以得到如下输出:

```
$ ls -F
Desktop/ Downloads/ Music/ Pictures/ Templates/ Videos/ Documents/ examples.desktop my_script* Public/ test_file
$
```

要把隐藏文件和普通文件及目录一起显示出来，就得用到-a参数。下面是一个带有-a参数 的ls命令的例子:

```
[admin@host-11-20-246-130 hupeidong]$ ls -a
.  ..  618  experiments_config  featureflow_op_repo  need_1  need_2  need_3  prediction_service_conf  serviceapi  tools
```

-R参数是ls命令可用的另一个参数，叫作递归选项。它列出了当前目录下包含的子目录中的文件。如果目录很多，这个输出就会很长。以下是-R参数输出的简单例子:

```
[admin@host-11-20-246-130 tools]$ ls -F -R
.:
batch_modify_config.py  opcenter_parser.py  predictor_cluster_statistic.sh  statistic/

./statistic:
exp1.txt  online1.txt  result_exp.txt  result_online.txt  result.txt  sort.txt
```

**窍门** 选项并不一定要像例子中那样分开输入:ls –F –R。它们可以进行如下合并:ls –FR。

-l参数会产生长列表格式的输出，包含了目录中每个文件的更多相关信息。

```
[admin@host-11-20-246-130 tools]$ ls -l
总用量 20
-rw-rw-r-- 1 admin admin 1539 5月  31 11:17 batch_modify_config.py
-rw-rw-r-- 1 admin admin 5156 4月  28 23:56 opcenter_parser.py
-rw-rw-r-- 1 admin admin 1488 5月  31 14:39 predictor_cluster_statistic.sh
drwxrwxr-x 2 admin admin 4096 5月  31 14:18 statistic
```

这种长列表格式的输出在每一行中列出了单个文件或目录。除了文件名，输出中还有其他有用信息。输出的第一行显示了在目录中包含的总块数。在此之后，每一行都包含了关于文件(或目录)的下述信息:
 文件类型，比如目录(d)、文件(-)、字符型文件(c)或块设备(b);
 文件的权限(参见第6章);
 文件的硬链接总数;
 文件属主的用户名;
 文件属组的组名;
 文件的大小(以字节为单位);
 文件的上次修改时间;
 文件名或目录名。

**过滤输出列表**

```
$ ls -l my_script
-rw-rw-r-- 1 admin admin 0 7月   9 18:37 my_script
$
```

ls命令能够识别标准通配符，并在过滤器中用它们进行模式匹配:
 问号(?)代表一个字符;
 星号(*)代表零个或多个字符。
问号可用于过滤器字符串中替代任意位置的单个字符。例如:

```
$ ls -l my_scr?pt
-rw-rw-r-- 1 admin admin 0 7月   9 18:37 my_scrapt
-rw-rw-r-- 1 admin admin 0 7月   9 18:37 my_script
```

星号可匹配零个或多个字符。

```
$ ls -l my*
-rw-rw-r-- 1 admin admin 0 7月   9 18:45 my_file
-rw-rw-r-- 1 admin admin 0 7月   9 18:37 my_scrapt
-rw-rw-r-- 1 admin admin 0 7月   9 18:37 my_script
$ ls -l my_s*t
-rw-rw-r-- 1 admin admin 0 7月   9 18:37 my_scrapt
-rw-rw-r-- 1 admin admin 0 7月   9 18:37 my_script
```

在过滤器中使用星号和问号被称为文件扩展匹配(file globbing)，指的是使用通配符进行模式匹配的过程。通配符正式的名称叫作元字符通配符(metacharacter wildcards)。除了星号和问号之外，还有更多的元字符通配符可用于文件扩展匹配。可以使用中括号。

```
$ ls -l my_scr[ai]pt
-rw-rw-r-- 1 christine christine 0 May 21 13:25 my_scrapt
-rwxrw-r-- 1 christine christine 54 May 21 11:26 my_script
$
```

在这个例子中，我们使用了中括号以及在特定位置上可能出现的两种字符:a或i。中括号表示一个字符位置并给出多个可能的选择。可以像上面的例子那样将待选的字符列出来，也可以指定字符范围，例如字母范围[a – i]。

```
$ ls -l f[a-i]ll
-rw-rw-r-- 1 christine christine 0 May 21 13:44 fall
-rw-rw-r-- 1 christine christine 0 May 21 13:44 fell
-rw-rw-r-- 1 christine christine 0 May 21 13:44 fill
$
```

另外，可以使用感叹号(!)将不需要的内容排除在外。

```
$ ls -l f[!a]ll
-rw-rw-r-- 1 christine christine 0 May 21 13:44 fell 
-rw-rw-r-- 1 christine christine 0 May 21 13:44 fill 
-rw-rw-r-- 1 christine christine 0 May 21 13:44 full 
$
```

### 4 处理文件

**4.1 创建文件**

touch命令还可用来改变文件的修改时间。这个操作并不需要改变文件的内容。

```
$ ls -l test_one
-rw-rw-r-- 1 christine christine 0 May 21 14:17 test_one 
$ touch test_one
$ ls -l test_one
-rw-rw-r-- 1 christine christine 0 May 21 14:35 test_one 
$
```

test_one文件的修改时间现在已经从最初的时间14:17更新到了14:35。如果只想改变访问时间，可用-a参数。

```
$ ls -l test_one
-rw-rw-r-- 1 christine christine 0 May 21 14:35 test_one 
$ touch -a test_one
$ ls -l test_one
-rw-rw-r-- 1 christine christine 0 May 21 14:35 test_one 
$ ls -l --time=atime test_one
-rw-rw-r-- 1 christine christine 0 May 21 14:55 test_one 
$
```

在上面的例子中，要注意的是，如果只使用ls –l命令，并不会显示访问时间。这是因为默认显示的是修改时间。要想查看文件的访问时间，需要加入另外一个参数:--time=atime。有了这个参数，就能够显示出已经更改过的文件访问时间。

**4.2 复制文件**

cp命令需要两个参数——源对象和目标对象: `cp source destination`

```
$ cp test_one test_two
$ ls -l test_*
-rw-rw-r-- 1 christine christine 0 May 21 14:35 test_one -rw-rw-r-- 1 christine christine 0 May 21 15:15 test_two 
$
```

新文件test_two和文件test_one的修改时间并不一样。如果目标文件已经存在，cp命令可能并不会提醒这一点。最好是加上-i选项，强制shell询问是否需要覆盖已有文件。

```
$ ls -l test_*
-rw-rw-r-- 1 christine christine 0 May 21 14:35 test_one -rw-rw-r-- 1 christine christine 0 May 21 15:15 test_two 
$
$ cp -i test_one test_two
cp: overwrite 'test_two'? n
$
```

也可以将文件复制到现有目录中。

```
$ cp -i test_one /home/christine/Documents/ 
$
$ ls -l /home/christine/Documents total 0
-rw-rw-r-- 1 christine christine 0 May 21 15:25 test_one 
$
```

新文件现就在目录Documents中了，和源文件同名。

之前的例子在目标目录名尾部加上了一个正斜线(/)，这表明Documents是目录而非文件。 这有助于明确目的，而且在复制单个文件时非常重要。如果没有使用正斜线，子目录 /home/christine/Documents又不存在，就会有麻烦。在这种情况下，试图将一个文件复制 到Documents子目录反而会创建一个名为Documents的文件，连错误消息都不会显示!

**上面说的就是，复制单个文件时，要加上正斜线(/)。**

cp命令的-R参数威力强大。可以用它在一条命令中递归地复制整个目录的内容。

```
$ ls -Fd *Scripts
Scripts/
$ ls -l Scripts/
total 25
-rwxrw-r-- 1 christine christine 929 Apr 2 08:23 file_mod.sh 
-rwxrw-r-- 1 christine christine 254 Jan 2 14:18 SGID_search.sh 
-rwxrw-r-- 1 christine christine 243 Jan 2 13:42 SUID_search.sh 
$
$ cp -R Scripts/ Mod_Scripts
$ ls -Fd *Scripts 
Mod_Scripts/ Scripts/ 
```

在执行cp –R命令之前，目录Mod_Scripts并不存在。它是随着cp –R命令被创建的，整个Scripts 目录中的内容都被复制到其中。

-d选项可能 还是第一次碰到。后者只列出目录本身的信息，不列出其中的内容。

也可以在cp命令中使用通配符。

```
$ cp *script Mod_Scripts/
$ ls -l Mod_Scripts
total 26
-rwxrw-r-- 1 christine christine 929 May 21 16:16 file_mod.sh 12 
-rwxrw-r-- 1 christine christine 54 May 21 16:27 my_script
-rwxrw-r-- 1 christine christine 254 May 21 16:16 SGID_search.sh
-rwxrw-r-- 1 christine christine 243 May 21 16:16 SUID_search.sh
$
```

该命令将所有以script结尾的文件复制到Mod_Scripts目录中。在这里，只需要复制一个文件: my_script。

**4.3 链接文件**

在Linux中有两种 不同类型的文件链接:
 符号链接 
 硬链接

使用ln命令以及-s选项来 创建符号链接。

```
$ ls -l data_file
-rw-rw-r-- 1 christine christine 1092 May 21 17:27 data_file 
$
$ ln -s data_file sl_data_file 
$
$ ls -l *data_file
-rw-rw-r-- 1 christine christine 1092 May 21 17:27 data_file
lrwxrwxrwx 1 christine christine 9 May 21 17:29 sl_data_file -> data_file 
$
```

符号链接的名字sl_data_file位于ln命令中的第二个参数位置上。

证明链接文件是独立文件的方法是查看inode编号。文件或目录的inode编号是一个用于标识的唯一数字，这个数字由内核分配给文件系统中的每一个对象。要查看文件或目录的inode 编号，可以给ls命令加入-i参数。

```
$ ls -i *data_file
296890 data_file 296891 sl_data_file 
$
```

硬链接会创建独立的虚拟文件，其中包含了原始文件的信息及位置。但是它们从根本上而言是同一个文件。

```
$ ls -l code_file
-rw-rw-r-- 1 christine christine 189 May 21 17:56 code_file 
$
$ ln code_file hl_code_file
$
$ ls -li *code_file
296892 -rw-rw-r-- 2 christine christine 189 May 21 17:56 code_file
296892 -rw-rw-r-- 2 christine christine 189 May 21 17:56 hl_code_file
$
```

**4.4 重命名文件**

在Linux中，重命名文件称为移动(moving)。

```
$ ls -li f?ll
296730 -rw-rw-r-- 1 christine christine 0 May 21 13:44 fall 296717 -rw-rw-r-- 1 christine christine 0 May 21 13:44 fell 294561 -rw-rw-r-- 1 christine christine 0 May 21 13:44 fill 296742 -rw-rw-r-- 1 christine christine 0 May 21 13:44 full 
$
$ mv fall fzll
$
$ ls -li f?ll
296717 -rw-rw-r-- 1 christine christine 0 May 21 13:44 fell 294561 -rw-rw-r-- 1 christine christine 0 May 21 13:44 fill 296742 -rw-rw-r-- 1 christine christine 0 May 21 13:44 full 296730 -rw-rw-r-- 1 christine christine 0 May 21 13:44 fzll 
$
```

也可以使用mv来移动文件的位置。

```
$ mv fzll Pictures/
```

也可以使用mv命令移动文件位置并修改文件名称，这些操作只需一步就能完成。

```
$ mv /home/christine/Pictures/fzll /home/christine/fall
```

**4.5 删除文件**

```
$ rm -i fall
rm: remove regular empty file 'fall'? y
$
$ ls -l fall
ls: cannot access fall: No such file or directory 
$
```

也可以使用通配符删除成组的文件。别忘了使用-i选项保护好自己的文件。

```
$ rm -i f?ll
rm: remove regular empty file 'fell'? y
rm: remove regular empty file 'fill'? y
rm: remove regular empty file 'full'? y
$
$ ls -l f?ll
ls: cannot access f?ll: No such file or directory 
$
```

### 5 处理目录

**5.1 创建目录**

要想同时创建多个目录和子目录，需要加入-p参数:

```
$ mkdir -p New_Dir/Sub_Dir/Under_Dir 
$
$ ls -R New_Dir 
New_Dir:
Sub_Dir

New_Dir/Sub_Dir:
Under_Dir

New_Dir/Sub_Dir/Under_Dir:
$
```

mkdir命令的-p参数可以根据需要创建缺失的父目录。父目录是包含目录树中下一级目录的目录。

**5.2 删除目录**

删除目录的基本命令是rmdir。

```
$ touch New_Dir/my_file 
$ ls -li New_Dir/
total 0
294561 -rw-rw-r-- 1 christine christine 0 May 22 09:52 my_file 
$
$ rmdir New_Dir
rmdir: failed to remove 'New_Dir': Directory not empty
$
```

默认情况下，rmdir命令只删除空目录。

一口气删除目录及其所有内容的终极大法就是使用带有-r参数和-f参数的rm命令。

```
$ tree Small_Dir
Small_Dir 7 ├─ a_file
├─ b_file
├─ c_file
├─ Teeny_Dir 8 │ └─e_file
└─ Tiny_Dir
      └─ d_file
2 directories, 5 files 
$
$ rm -rf Small_Dir
$
$ tree Small_Dir
Small_Dir [error opening dir]
0 directories, 0 files
$
```

rm -rf命令既没有警告信息，也没有声音提示。这肯定是一个危险的工具，尤其是在拥有超级用户权限的时候。务必谨慎使用，请再三检查你所要进行的操作是否符合预期。

### 6 查看文件内容

**6.1 查看文件类型**

file命令是一个随手可得的便捷工具。它能够探测文件的内部，并决定文件是什么类型的:

```
$ file my_file my_file: ASCII text 
$
```

以后可以使用file命令作为另一种区分目录的方法:

```
$ file New_Dir New_Dir: directory 
$
```

**6.2 cat命令**

-n参数会给所有的行加上行号。

```
$ cat -n test1
     1	hello
     2
     3	This is a test file.
     4
     5
     6	That we'll use to   test the cat command.
```

如果只想给有文本的行加上行号，可以用-b参数。

```
$ cat -b test1
     1	hello

     2	This is a test file.


     3	That we'll use to   test the cat command.
```

如果不想让制表符出现，可以用-T参数。-T参数会用^I字符组合去替换文中的所有制表符。

```
$ cat -T test1
hello

This is a test file.


That we'll use to   test the cat command.
```

**6.3 more命令**

more命令只支持文本文件中的基本移动。如果要更多高级功能，可以试试less命令。

**6.4 less命令**

其中一组特性就是less命令能够识别上下键以及上下翻页键(假设你的终端配置正确)。在查看文件内容时，这给了你全面的控制权。

**6.5 tail命令**

tail命令浏览文件最后10行的效果:

```
$ tail onhouses
你们不应为穿过房门而收敛翅膀，不应为防止撞到天花板而俯身低头，也不应因担心墙壁开裂坍塌而屏住呼吸。

You shall not dwell in tombs made by the dead for the living.
你们不应居住在死者为生者建造的坟墓中。

And though of magnificence and splendour, your house shall not hold your secret nor shelter your longing.
纵然你们的宅邸金碧辉煌，它们也无法隐藏你们的秘密，掩盖你们的愿望。

For that which is boundless in you abides in the mansion of the sky, whose door is the morning mist, and whose windows are the songs and the silences of night.
因为你们内在的无穷性居住在天宫里，它以晨雾为门，以夜的歌声和寂静为窗。
```

通过加入-n 2使 tail命令只显示文件的最后两行:

```
$ tail -n 2 onhouses
For that which is boundless in you abides in the mansion of the sky, whose door is the morning mist, and whose windows are the songs and the silences of night.
因为你们内在的无穷性居住在天宫里，它以晨雾为门，以夜的歌声和寂静为窗。
```

-f参数是tail命令的一个突出特性。它允许你在其他进程使用该文件时查看文件的内容。tail命令会保持活动状态，并不断显示添加到文件中的内容。这是实时监测系统日志的绝妙方式。

**6.6 head命令**

显示文件前10行 的文本:

```
$ head log_file line1
line2
line3
line4
line5
Hello World - line 6
line7
line8
line9
line10
$
```

类似于tail命令，它也支持-n参数，这样就可以指定想要显示的内容了。这两个命令都允许你在破折号后面输入想要显示的行数:

```
$ head -5 log_file line1
line2
line3
line4 line5 
$
```

## 第 4 章 更多的bash shell命令

### 1 监测程序

**探查进程**

默认情况下，ps命令并不会提供那么多的信息:

```
[admin@host-11-20-246-130 centos]$ ps
   PID TTY          TIME CMD
457076 pts/1    00:00:00 bash
508142 pts/1    00:00:00 ps
```

默认情况下，ps命令只会显示运行在当前控制台下的属于当前用户的进程。在此例中，我们只运行了bash shell(注意，shell也只是运行在系统上的另一个程序而已)以及ps命令本身。

举个例子，如果你想查看系统上运行的所有进程，可用-ef参数组合

```
-e 显示所有进程
-f 显示完整格式的输出
```

```
[admin@host-11-20-246-130 centos]$ ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0  2019 pts/4    00:00:37 /bin/sh -c tar2export && /usr/sbin/sshd && /usr/sbin/crond && sleep 9999999d
root         55      1  0  2019 ?        00:00:03 /usr/sbin/sshd
root         57      1  0  2019 ?        00:16:55 /usr/sbin/crond
root         58      1  0  2019 pts/4    00:00:00 sleep 9999999d
```

如果想要获得更多的信息，可采用-l参数，它会产生一个长格式输出。

```
$ ps -l
F S UID PID PPID C PRI NI ADDR SZ WCHAN TTY TIME CMD 
0 S 500 3081 3080 0 80 0 - 1173 wait pts/0 00:00:00 bash 
0 R 500 4463 3081 1 80 0- 1116- pts/0 00:00:00ps
$
```

**实时监测进程**

top

```
top - 20:15:11 up 982 days,  9:08,  0 users,  load average: 10.06, 8.48, 7.59
Tasks:  64 total,   1 running,  63 sleeping,   0 stopped,   0 zombie
%Cpu(s):  8.3 us,  1.9 sy,  0.0 ni, 89.7 id,  0.0 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem : 39570720+total, 17873705+free, 54948260 used, 16202190+buff/cache
KiB Swap:        0 total,        0 free,        0 used. 30423516+avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
311784 admin     20   0 1143888 164400  18356 S   1.3  0.0   7:02.18 node
297171 admin     20   0 6685196  20952    156 S   0.3  0.0  27:43.45 cpptools
519046 admin     20   0  144036   1944   1364 R   0.3  0.0   0:00.02 top
     1 root      20   0   11624    228     32 S   0.0  0.0   0:37.05 sh
    55 root      20   0   82540   1048    196 S   0.0  0.0   0:03.56 sshd
    57 root      20   0   22776   1020    352 S   0.0  0.0  16:55.84 crond
    58 root      20   0    4308     88      0 S   0.0  0.0   0:00.00 sleep
 25370 admin     20   0  440524   6796    988 S   0.0  0.0 156:41.99 deploy
 44425 admin     20   0  281720   9512   1308 S   0.0  0.0 778:25.24 python
```

**结束进程**

kill命令可通过进程ID(PID)给进程发信号。默认情况下，kill命令会向命令行中列出的全部PID发送一个TERM信号。遗憾的是，你只能用进程的PID而不能用命令名，所以kill命令有时并不好用。

```
$ kill 3940
-bash: kill: (3940) - Operation not permitted $
```

TERM信号告诉进程可能的话就停止运行。不过，如果有不服管教的进程，那它通常会忽略这个请求。

killall命令

killall命令非常强大，它支持通过进程名而不是PID来结束进程。killall命令也支持通配符，这在系统因负载过大而变得很慢时很有用。
```
# killall http*
#
```

上例中的命令结束了所有以http开头的进程，比如Apache Web服务器的httpd服务。

### 2 监测磁盘空间

**使用df命令**

**使用du命令**

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****


****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****



****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****



****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****



****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****



****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****



****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****

****
