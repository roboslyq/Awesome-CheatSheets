[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheets)

> 📌 相较于这些参考资料本文希望能够更为生动详细，并且不仅仅局限于 Bash 本身，而是包含具有一定价值的其他扩展命令，从而更贴近于日常工作中的需要。

# Linux Commands CheatSheet | 常用命令与技巧清单

Shell 即是用户和 Linux 内核之间的接口程序，其可以被看做命名语言解释器（Command-Language Interpreter）。Bash(Bourne Again Shell) 则是 Bourne Shell(Sh) 的扩展，其优化了原本用户输入处理的不足，提供了多种便捷用户输入的方式。

# 命令执行

# 文件系统

## 文件检索

使用 `ls -l` 查看目录下文件列表如统计 /home/han 目录 ( 包含子目录 ) 下的所有 js 文件则:

```sh
$ ls -lR /home/han | grep js | wc -l

$ ls -l "/home/han" | grep "js" |wc -l

$ ls -l --sort=size --block-size=M
```

这类似于 SQL 中的 % 符号，例如，使用 `WHERE first_name LIKE 『John%` 搜索所有以 John 起始的名字。在 Bash 中，相应的命令是`John*`。如果想列出一个文件夹中所有以 `.json` 结尾的文件，可以输入 `ls *.json`。

可以使用 [fzf](https://github.com/junegunn/fzf) 进行交互式检索，在[这里](https://github.com/junegunn/fzf-bin/releases)下载二进制文件

```sh
# 根据文件类型搜索
$ find * -type f | fzf > selected

# 根据文件名匹配
$ find . -name '*.map' -exec rm {} \;

# 批量修改文件的权限
$ find /opt/lampp/htdocs -type f -exec chmod 644 {} \;
```

## 文件操作

```shell
# 创建文件夹
mkdir <name>

# 递归创建父文件夹
mkdir -p / --parents backup/old

# 创建文件夹时同时指定权限
mkdir -m a=rwx backup

mkdir -p -m 777 backup/server/2011/11/30
```

```sh
# 将文件解压到指定文件名
$ tar -xvzf fileName.tar.gz -C newFileName

# 将文件夹压缩到指定目录
$ tar -czf target.tar.fz file1 file2 file3

# 指定文件名创建压缩包
$ tar -cv -f archive.tar file1.txt file2.txt

# 过滤某个文件夹下面的某些子文件夹
$ tar cfvz --exclude='<dir1>' --exclude='<dir2>' target.tgz target_dir
$ tar cvf dir.tar.gz --exclude='/dir/subdir/subsubdir/*' dir

# 向压缩包中添加文件
$ tar -rf archive.tar file3.txt
```

有时候我们还需要将大型文件进行切割处理：

```sh
$ split -b 10M home.tar.bz2 "home.tar.bz2.part"
$ ls -lh home.tar.bz2.parta*
# 混合 tar 命令使用
$ tar -cvzf - wget/* | split -b 150M - "downloads-part"

# 拼接文件
$ cat home.tar.bz2.parta* >backup.tar.gz.joined
```

## 分区与挂载

fdisk 常用于进行磁盘分区，主分区（包含扩展分区）、逻辑分区，主分区最多有 4 个（包含扩展分区）。因此我们在对硬盘分区时最好划分主分区连续，比如说：主分区一、主分区二、扩展分区。

```sh
# 查看系统上的硬盘
$ fdisk -l

# 操作指定磁盘
$ fdisk /dev/sda

# 列出当前操作硬盘的分区情况
$ p

# 删除某个分区
$ d

# 增加某个分区
$ n
```

`mkfs.ext4` 等常用于对分区进行格式化，mount 则是用于挂载分区：

```sh
# 格式化
$ mkfs.ext4 /dev/xvdb1

# 挂载
$ mount /dev/xvdb1 /data2

# 卸载
$ umount /data2
```

## 磁盘 IO

使用 du(disk usage)/df(disk free) 查看磁盘状态，du 是通过搜索文件来计算每个文件的大小然后累加，du 能看到的文件只是一些当前存在的，没有被删除的。他计算的大小就是当前他认为存在的所有文件大小的累加和；df 记录的是通过文件系统获取到的文件的大小，他比 du 强的地方就是能够看到已经删除的文件，而且计算大小的时候，把这一部分的空间也加上了，更精确了。

```sh
# 查看磁盘剩余空间
$ df -ah

$ df --block-size=GB/-k/-m

# 查看当前目录下的目录空间占用
$ du -h --max-depth=1 /var/ | sort

# 查看 tmp 目录的磁盘占用
$ du -sh /tmp

# 查看当前目录包含子目录的大小
$ du -sm .

# 查看目录下文件尺寸
$ ls -l --sort=size --block-size=M
```

使用 `fuser tmpFile.js` 查看指定文件被进程占用情况，

# 文本处理

## tail & head & cat & more

`tailf` 命令类似于 `tail -f`，其可以打印出文件的最后十行内容，并且会随着文件的增长而自动滚动；不过其不会在文件没有变化的时候去频繁访问文件。

```sh
$ head -5 /etc/passwd

$ tail -10 /etc/passwd
$ tail -n 10 /etc/passwd
$ tail -f /var/log/messages

# 查看文件中间的第 5-10 行
$ sed -n '5,10p' /etc/passwd
```

## grep & ack

grep（global search regular expression(RE) and print out the line，全面搜索正则表达式并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

```sh
# 基本检索
$ grep match_pattern file_name
$ grep "match_pattern" file_1 file_2 file_3 ...

# 仅显示不匹配的文本
$ grep -v "match_pattern" file_1 file_2 file_3 ...

# 搜索多个文件并查找匹配文本在哪些文件中
$ grep -l "text" file1 file2 file3...

# 递归检索
$ grep "text" . -r -n
#只在目录中所有的.php和.html文件中递归搜索字符"main()"
$ grep "main()" . -r --include *.{php,html}
#在搜索结果中排除所有README文件
$ grep "main()" . -r --exclude "README"
#在搜索结果中排除filelist文件列表里的文件
$ grep "main()" . -r --exclude-from filelist

# 正则表达式检索
$ grep -E "[1-9]+"
$ egrep "[1-9]+"

# 检索当前目录下
$ grep 'John Resig' *

# 只输出文件中匹配到的部分 -o 选项
$ echo this is a test line. | grep -o -E "[a-z]+\."
# 输出模板在文本中匹配行数
$ grep -c "text" file_name
# 输出字符的总匹配行数
$ grep "text" file_name | wc -l
# 忽略字符串大小写
$ echo "hello world" | grep -i "HELLO"

# 显示匹配某个结果之后的3行，使用 -A 选项，-B 显示匹配某个结果之前的3行， -C 显示匹配某个结果的前三行和后三行
$ seq 10 | grep "5" -A 3

# 与其他命令协同使用
$ grep foo $(find . -name '*.pm' | grep -v .svn)
$ ls -l | grep .py
```

[ag](https://github.com/ggreer/the_silver_searcher) 是类似于 ack 但是性能更优地工具。

```sh
# ack 默认进行递归搜索
$ ack hello
$ ack -i hello
$ ack -v hello
$ ack -w hello
$ ack -Q 'hello*'

# 输出文件指定内容
# 输出所有文件第二行
$ ack --line=1

# 也可以指定目录/文件名/文件类型检索
# 查找所有python文件
$ ack --python hello
# 查找匹配正则的文件
$ ack -G hello.py$ hello

# ack 能够对搜索结果进行方便地定制
# 仅显示包含的文件名
$ ack -l 'hello'
# 仅显示非包含文件名
$ ack -L 'print'
# 以less形式展示
$ ack hello --pager='less -R'
# 不在头上显示文件
$ ack hello --noheading
# 不对匹配字符着色
$ ack hello --nocolor

# ack 同样能够用于查找文件
# 查找全匹配文件
$ ack -f hello.py
# 查找正则匹配文件
$ ack -g hello.pyf
#查找然后排序
$ ack -g hello --sort-files
```

# 用户权限

## SSH

```sh
# 生成名为 id_rsa 的私钥文件和名为 id_rsa.pub 的公钥文件
$ ssh-keygen -t rsa

# 指定 4096 位的长度
$ ssh-keygen -b 4096 -t rsa

# 使用 ssh-copy-id 添加公钥
ssh-copy-id username@remote-server
```

生成的 id_rsa.pub 公钥文件也可以用于配置 Git 仓库的 SSH 访问等。我们可以使用 ssh 登录到本机(切换用户)或者远端 Linux 设备中，通过将本机生成的公钥文件写入目标机器的 authorized_keys 即可以实现免密码登录。

## 用户管理

将 shell 切换为其他用户，使用 `su username` 或者 `sudo - username`，加入 `-` 会使得切换后的环境与使用该用户登录后的环境相同，省略用户名则默认为 root。

## 权限设置

```sh
$ find /opt/lampp/htdocs -type f -exec chmod 644 {} \;
```

# 系统进程

## 系统基础

## 运行状态

## 系统服务

systemctl status 命令用于查看系统状态和单个 Unit 的状态。

```sh
# 显示系统状态
$ systemctl status

# 显示单个 Unit 的状态
$ sysystemctl status bluetooth.service

# 显示远程主机的某个 Unit 的状态
$ systemctl -H root@rhel7.example.com status httpd.service
```

我们最常用的就是 Unit 管理命令：

```sh
# 立即启动一个服务
$ sudo systemctl start apache.service

# 立即停止一个服务
$ sudo systemctl stop apache.service

# 重启一个服务
$ sudo systemctl restart apache.service

# 杀死一个服务的所有子进程
$ sudo systemctl kill apache.service

# 重新加载一个服务的配置文件
$ sudo systemctl reload apache.service

# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload

# 显示某个 Unit 的所有底层参数
$ systemctl show httpd.service

# 显示某个 Unit 的指定属性的值
$ systemctl show -p CPUShares httpd.service

# 设置某个 Unit 的指定属性
$ sudo systemctl set-property httpd.service CPUShares=500
```

## 系统检视

### 版本型号

- 使用 `hostname` 查看当前主机名，使用 `sudo hostname newName` 修改当前主机名

```sh
# 显示当前主机的信息
$ hostnamectl

# 设置主机名。
$ sudo hostnamectl set-hostname rhel7
```

- 查看 Linux 系统版本

```bash
# 查看内核版本
$ cat /proc/version
Linux version 2.6.18-238.el5 (mockbuild@x86-012.build.bos.redhat.com) (gcc version 4.1.2 20080704 (Red Hat 4.1.2-50)) #1 SMP Sun Dec 19 14:22:44 EST 2010

$ uname -r
2.6.18-238.el5

$ uname -a
Linux SOR_SYS.99bill.com 2.6.18-238.el5 #1 SMP Sun Dec 19 14:22:44 EST 2010 x86_64 x86_64 x86_64 GNU/Linux

# 查看 Linux 发行版本
$ lsb_release -a

$ cat /etc/issue

Red Hat Enterprise Linux Server release 5.6 (Tikanga)
Kernel \r on an \m

$ file /bin/bash

/bin/bash: ELF 64-bit LSB executable, AMD x86-64, version 1 (SYSV), for GNU/Linux 2.6.9, dynamically linked (uses shared libs), for GNU/Linux 2.6.9, stripped
```

### 运行状态

我们可以使用 `uptime` 或者 `w` 来查看当前用户的接入时间：

```sh
$ w

# 15:33:49 up 58 days,  5:45,  1 user,  load average: 0.12, 0.15, 0.22
# USER     TTY        LOGIN@   IDLE   JCPU   PCPU WHAT
# root     pts/1     15:15   49.00s  0.04s  0.00s w
```

查看系统当前的 CPU 与内存情况：

```sh
$ lscpu
```

## 进程监控

- 使用 `top` 查看进程资源占用情况，也可以使用扩展 htop 或者 gtop；如果针对容器监控，可以使用 [ctop](https://github.com/bcicen/ctop)。

- 使用 `pstree -p` 查看当前进程树，使用 `ps -A` 查看所有进程信息，使用 `ps -aux` 查看所有正在内存中的程序，使用 `ps -ef` 查看所有连同命令行的进程信息；使用 `ps -u root` 显示指定用户信息；使用 `ps -ef | grep ssh` 查看特定进程。

## 进程管理

```sh
$ pkill -f java
```

ulimit 命令用来限制系统用户对 shell 资源的访问。ulimit 用于限制 shell 启动进程所占用的资源，支持以下各种类型的限制：所创建的内核文件的大小、进程数据块的大小、Shell 进程创建文件的大小、内存锁住的大小、常驻内存集的大小、打开文件描述符的数量、分配堆栈的最大大小、CPU 时间、单个用户的最大线程数、Shell 进程所能使用的最大虚拟内存。同时，它支持硬资源和软资源的限制。

我们常用的就是在 Web 服务器上修改每个进程可以打开的最大文件数：

```sh
$ ulimit -n 4096 # 将每个进程可以打开的文件数目加大到4096，缺省为1024
```

其他建议设置成无限制（unlimited）的一些重要设置是：

```sh
$ ulimit -d unlimited # 数据段长度
$ ulimit -m unlimited # 最大内存大小
$ ulimit -s unlimited # 堆栈大小
$ ulimit -t unlimited # CPU 时间
$ ulimit -v unlimited # 虚拟内存
```

我们也可以通过配置文件的方式永久写入：

```sh
vi /etc/security/limits.conf
# 添加如下的行
* soft noproc 11000
* hard noproc 11000
* soft nofile 4100
* hard nofile 4100
# 说明：* 代表针对所有用户，noproc 是代表最大进程数，nofile 是代表最大文件打开数
```

## Cron

![](http://fs.gimoo.net/img/2014/10/12/011835_5439666b84167.jpg)

```sh
\*　　 \*　　 \*　　 \*　　 \*　　 command
分　   时　   日　   月　   周　   命令
```

第 1 列表示分钟 1 ～ 59 每分钟用*或者 */1 表示第 2 列表示小时 1 ～ 23 ( 0 表示 0 点)第 3 列表示日期 1 ～ 31 第 4 列表示月份 1 ～ 12 第 5 列标识号星期 0 ～ 6 ( 0 表示星期天)第 6 列要运行的命令

在以上各个字段中，还可以使用以下特殊字符：

星号(\* )：代表所有可能的值，例如 month 字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。

逗号(, )：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”

中杠(- )：可以用整数之间的中杠表示一个整数范围，例如 “2-6” 表示 “2,3,4,5,6”

正斜线(/ )：可以用正斜线指定时间的间隔频率，例如 “0-23/2” 表示每两小时执行一次。同时正斜线可以和星号一起使用，例如 \*/10，如果用在 minute 字段，表示每十分钟执行一次。

cron 是一个可以用来根据时间、日期、月份、星期的组合来调度对重复任务的执行的守护进程。

cron 假定系统持续运行。如果当某任务被调度时系统不在运行，该任务就不会被执行。√√

# 网络

## 状态检索

```sh
# 查看指定端口的占用情况
$ lsof -i:80
$ kill -9 $(lsof -t -i:8080)
# 查看某个进程的 TCP 连接
$ lsof -p <pid> | grep TCP

# 查看 TCP 连接数
$ netstat -an
# 统计80端口连接数
$ netstat -nat|grep -i "80"|wc -l
# 统计 IP 地址连接数
netstat -na|grep ESTABLISHED|awk {print $5}|awk -F: {print $1}| sort |uniq -c|sort -r +0n

netstat -na|grep SYN|awk {print $5}|awk -F: {print $1}|sort|uniq -c|sort -r +0n
```

## 网络配置

## 网络请求

```sh
# 展示 curl 的进度
$ curl --progress-bar -T "${SOME_LARGE_FILE}" "${UPLOAD_URL}" | tee /dev/null
```

# Todos

- [ ] [the-art-of-command-line](https://parg.co/bXZ)
- [ ] [Linux Commands Cheat Sheet](https://parg.co/Uqu)
