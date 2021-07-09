# 1 SSH

## 1.1 生成 SSH 公钥

如前所述，许多 Git 服务器都使用 SSH 公钥进行认证。为了向 Git 服务器提供 SSH 公钥，如果某系统用户尚未拥有密钥，必须事先为其生成一份。这个过程在所有操作系统上都是相似的。首先，你需要确认自己是否已经拥有密钥。默认情况下，用户的 SSH 密钥存储在其 ~/.ssh 目录下。进入该目录并列出其中内容，你便可以快速确认自己是否已拥有密钥：

```
$ cd ~/.ssh
$ ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```

我们需要寻找一对以 id_dsa 或 id_rsa 命名的文件，其中一个带有 .pub 扩展名。.pub 文件是你的公钥，另一个则是与之对应的私钥。如果找不到这样的文件（或者根本没有 .ssh 目录），你可以通过运行 ssh-keygen 程序来创建它们。

```
$ ssh-keygen -o
Generating public/private rsa key pair.
Enter file in which to save the key (/home/schacon/.ssh/id_rsa):
Created directory '/home/schacon/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/schacon/.ssh/id_rsa.
Your public key has been saved in /home/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
d0:82:24:8e:d7:f1:bb:9b:33:53:96:93:49:da:9b:e3 schacon@mylaptop.local
```

首先 ssh-keygen 会确认密钥的存储位置（默认是 .ssh/id_rsa），然后它会要求你输入两次密钥口令。如果你不想在使用密钥时输入口令，将其留空即可。然而，如果你使用了密码，那么请确保添加了 -o 选项，它会以比默认格式更能抗暴力破解的格式保存私钥。你也可以用 ssh-agent 工具来避免每次都要输入密码。

现在，进行了上述操作的用户需要将各自的公钥发送给任意一个 Git 服务器管理员 （假设服务器正在使用基于公钥的 SSH 验证设置）。他们所要做的就是复制各自的 .pub 文件内容，并将其通过邮件发送。公钥看起来是这样的：

```
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU
GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3
Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA
t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En
mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx
NrRFi9wrf+M7Q== schacon@mylaptop.local
```

## 1.2 使用记录

尽量把 id_rsa 文件保存在 ~/.ssh/ 目录下面，改个后缀。

```
admin@host-172-28-34-29 ~ $ ssh-add ~/.ssh/id_rsa_hpd
Identity added: /home/admin/.ssh/id_rsa_hpd (/home/admin/.ssh/id_rsa_hpd)
admin@host-172-28-34-29 ~ $ cat ~/.ssh/id_rsa_hpd.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC8OOQAF7FE8W1xYmuQjjUZ542BmVPJouW9cUAl+ievHoLfo8yocB9JaCaC3cJInoixbj/1/AkoxpFefsGe90RU3D04zWogV9AARUC9hgPcctEKWIJlypr14c1W99nL53yIQR552vaH8cFCpwynYRWY+QJI+HfjDUiLcTbmDf12/aZVruFUfrL087RXyXymw5w5PZKk0Rs+C5gshHed/j0G92UDqCE9zgfSUA0dQaTfwncgjZ5kQaoZ1VC3tvqqAC5vWH+cBMMA6FKqc6iTsIKE/3iyKORSUNmakEkayYcEI04YevVxxveY6HHG1PgOf8Qkc2oR/YSBihiZnPZOu58V admin@host-172-28-34-29
```

如果执行 ssh-add 时出现 **Could not open a connection to your authentication agent** 在执行 ssh-add ~/.ssh/id_ras 时发生此错，

执行如下命令　`ssh-agent bash`
然后再执行 `ssh-add ~/.ssh/id_ras` 即可。

## 1.3 笔记

1. 生成自己的公钥和私钥

```
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_hupeidong -C "hupeidong@jd.com"

$ ssh-agent bash (如果下面命令报错，则执行此命令)

$ ssh-add ~/.ssh/id_rsa_hupeidong
```

在 git 和 gerrit 中添加生成的 `id_rsa_hupeidong.pub`

# 2 Pending Cache

可以建立一个 Pending Cache 服务，用于存放一个分片计算 common 数据的结果，其他分片则等待这个分片计算完成并且取到计算结果。取结果需要 rpc 调用。由于 prepare 和 取数是并行的，且取数的耗时更长，所以对于等待计算结果的机器也不会有很大的影响，而且能够减少 CPU。

