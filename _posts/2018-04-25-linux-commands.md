---
layout: post
title:  "Linux 基础命令"
date:   2018-04-25 18:58:02 +0800
author: mistydew
categories: Unix/Linux
tags: Linux CLI Terminal
---
Linux mascot: crystal tux and tux(Torvalds linUX?)<br>
![crystal tux](/images/20180425/crystal_penguin.jpg)
![tux](/images/20180425/tux.jpg)<br>
Childhood and Adult(funny)

## 切换目录 change directory

{% highlight shell %}
$ cd <dir> # 切换工作目录（当前所在目录）至 <dir> 目录。
$ cd .. # 切换工作目录至上一级目录。
$ cd - # 切换工作目录至上一个工作目录。
$ cd ~ # 同 `$ cd` 切换工作目录至当前用户家目录。
{% endhighlight %}

**注：本文出现的 `<dir>` 均为相对路径 或 绝对路径**

## 查看目录下的内容 list

{% highlight shell %}
$ ls # 列出当前所在目录下的所有文件（除隐藏文件，即以 `.` 开头的文件）的文件名，包含普通文件、目录文件...
$ ls <dir> # 列出 <dir> 目录下的所有文件（除隐藏文件，即以 `.` 开头的文件）的文件名，包含普通文件、目录文件...
$ ls -l # 列出当前所在目录下的所有文件（除隐藏文件）的详细信息，分别是文件属性（类型、权限）、连接数、所属者、所属组、大小（字节）、日期、文件名。
$ ls -a # 列出当前所在目录下的所有文件的文件名，包含隐藏文件。
$ ls -R # 列出当前所在目录及其子目录下的所有文件的文件名。
{% endhighlight %}

## 创建文件 make

{% highlight shell %}
$ touch <file> # 创建一个指定文件名为 <file> 的空文件。
$ mkdir <dir> # 创建一个指定目录名为 <dir> 的空目录。
$ echo <content> >> <file> # 追加 <content> 内容到 <file> 文件的最后一行，若文件不存在，则新建并追加。注：使用 `>` 代替 `>>`，则会覆盖现存文件当前的内容。`>>` 为输出重定向。
$ vi/vim <file> # 使用 Vi/Vim 编辑器打开指定的文件进行编辑，若文件不存在，则新建空文件再进行编辑。
{% endhighlight %}

**注：1. Vi/Vim 编辑过的文件会在结尾自动添加一个 `\n` 换行，所以编辑过文件至少又一个字节的大小。2. Vi 系统自带，Vim 需安装，详见安装软件命令。**

## 删除文件 remove

{% highlight shell %}
$ rm <file> # 删除指定的 <file> 文件。
$ rmdir <dir> # 只能删除指定的空目录 <dir>。
$ rm <dir> -r # 删除指定的目录 <dir> 下包含的所有文件，`-r` 参数表示循环。
{% endhighlight %}

**注：慎用 `$ sudo rm / -rf` # 删除根目录下所有文件，使用 `sudo` 进行权限提升，`-f` 参数表示不显示任何信息。
详见[多一个空格会发生什么？](https://github.com/MrMEEE/bumblebee-Old-and-abbandoned/commit/6cd6b2485668e8a87485cb34ca8a0a937e73f16d)**

## 安装软件 install

{% highlight shell %}
$ apt-get install vim # 安装 vim 软件到系统，一般默认装在 `/usr/bin` 目录下，所以要使用 root 用户进行安装，或在命令的前面加 `sudo` 进行权限提升。e.g. `$ sudo apt-get install vim`。
$ apt-get update # 从所有配置的源中下载包信息，用于安装软件时显示没有该软件的情况。
$ apt-get upgrade # 从通过 sources.list 配置的源安装在系统上的所有包中安装可用的升级，即通过 update 的结果进行已安装软件的升级。
{% endhighlight %}

## 查看磁盘

{% highlight shell %}
$ du -sh # 查看当前所在目录下各文件的总大小，参数 `-s` 同 `--summarize` 表示总和，参数 `-h` 同 `--human-readable` 表示以当前大小可表示的最大单位来显示信息，若不加该参数，则单位默认为 KB。
$ du -h -d 1 # 查看当前所在目录下深度为 1 的各文件的总大小，参数 `-d N` 同 `--max-depth=N` 表示显示的目录层级的最大深度为 N。
$ df -h # 查看当前磁盘各分区的使用情况，参数 `-h` 同 `--human-readable` 含义同上。
$ fdisk -l # 查看当前磁盘及其分区的详细信息。注：若不显示任何信息，命令前需加 `sudo` 来提升权限。
{% endhighlight %}

## 远程安全拷贝 secure copy

{% highlight shell %}
$ scp <file> username@ip:<dir> # 复制本地文件 <file> 到远程主机 `username@ip` 的 <dir> 目录下。
$ scp username@ip:<file> <dir> # 复制远程主机 `username@ip` 下的文件 <file> 到本地 <dir> 目录下。
{% endhighlight %}

**注：`username` 和 `ip` 的顺序，且需要知道 `username` 对应的密码。**

## 统计时间 time

{% highlight shell %}
$ time <command> # 统计执行给定命令 <command> 所花费的时间。
{% endhighlight %}

## 查找文件 find

{% highlight shell %}
$ find . -name <filename> -type f # 以当前目录 `.` 为起始目录，查询并显示指定文件名 <filename> 的文件（相对）路径，`f` 表示文件类型普通文件。
$ find . -empty #  以当前目录 `.` 为起始目录，查询并显示所有大小为 0 的文件（相对）路径。
{% endhighlight %}

## 搜索内容 grep

{% highlight shell %}
$ grep <content> * -nri # 查询并显示当前所在目录下所有出现 <content> 字符串的文件及行号，`*` 表示当前目录下所有文件，`-n` 参数显示所在行号，`-r` 参数表示递归，`-i` 参数表示不区分大小写。
$ grep <content> <file> -n # 查询并显示指定文件 <file> 出现 <content> 字符串的行号。
{% endhighlight %}

## 更改文件所属用户（或组） change owner

{% highlight shell %}
$ chown <username> <file> # 改变文件 <file> 所属用户名 <username>。
$ chown <username>:<group> <file> # 改变文件 <file> 所属用户名 <username> 或组 <group>。
{% endhighlight %}

**注：只有文件创建者和 root 用户才能使用此命令，且 `<username>` 必须是本机存在的用户名。<br>
本机用户名记录在 `/etc/passwd` 文件中。**

## 更改文件所属组 change group

{% highlight shell %}
$ chgrp <group> <file> # 改变文件 <file> 所属组 <group>。
{% endhighlight %}

**注：只有文件创建者和 root 用户才能使用此命令，且 `<group>` 必须是本机存在的组。<br>
本机用户所属组记录在 `/etc/group` 文件中。**

## 发信号到进程 kill

{% highlight shell %}
$ kill -l # 列出全部信号。
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
$ kill -9 <pid> # 杀死指定 <pid> 的进程。
$ kill -19 <pid> # 暂停指定进程 <pid>，和 `Ctrl+Z` 效果相同。
{% endhighlight %}

## 进程的前后台切换 & jobs fg Ctrl+Z bg

{% highlight shell %}
$ ./<ELF> & # 后加 `&` 符号可把程序 <ELF> 放后台运行。
$ jobs # 可查看后台运行的程序，包含 `Ctrl+Z` 暂停的进程，及其序号和运行状态。
$ fg n # 把 `jobs` 列出的后台序号为 `n` 的任务拉到前台运行。
$ Ctrl + Z # 把当前前台运行的程序放到后台并挂起。
$ bg n # 把 `jobs` 列出的后台序号为 `n` 的任务继续运行。
{% endhighlight %}

## 查看打开文件的进程 list open file

{% highlight shell %}
$ lsof -i[:<port>] # 查看占用端口 <port> 的进程，注意 `sudo`。
{% endhighlight %}

## 解压缩文件 tar

{% highlight shell %}
$ tar zcfv <filename>.tar.gz <filename> # 压缩文件 <filename> 为 gz 格式 `<filename>.tar.gz`。
$ tar xfv <filename>.tar.gz # 解压缩 gz 格式的文件 `<filename>.tar.gz`。
$ tar Jcfv <filename>.tar.xz <filename> # 压缩文件 <filename> 为 xz 格式 `<filename>.tar.xz`。
$ tar Jxfv <filename>.tar.xz # 解压缩 xz 格式的文件 `<filename>.tar.xz`。
{% endhighlight %}

## 创建连接文件 link

{% highlight shell %}
$ ln <srcfile> <destfile> # 创建源文件 <srcfile> 的硬链接 <destfile>，删除源文件不影响其硬链接文件。<br>
$ ln -s <srcfile> <destfile> # 创建源文件 <srcfile> 的软链接 <destfile>，类似于 `Windows` 下的快捷方式，删除源文件后其软链接文件失效。
{% endhighlight %}

## 统计文件内容 wc

{% highlight shell %}
$ wc -l <filename> # 统计文件 <filename> 的行数。参数 `-c` 统计字符数即文件大小字节数，`-w` 统计字数。
{% endhighlight %}

## 统计文件个数 ls | wc

{% highlight shell %}
$ ls -l | grep "^-" | wc -l # 统计当前目录下普通文件个数。
$ ls -l | grep "^d" | wc -l # 统计当前目录下目录文件个数。
$ ls -lR | grep "^-" | wc -l # 统计当前目录及其子目录下普通文件个数。
$ ls -lR | grep "^d" | wc -l # 统计当前目录及其子目录下目录文件个数。
{% endhighlight %}

## 查看文件 cat

{% highlight shell %}
$ cat -A <file> # 把文件 <file> 内容全部输出至标准输出，`-A` 用于显示不可打印字符，行尾换行显示为 `$`。
$ more <file> # 可滚动翻看文件 <file> 的内容，`Space` 键查看下一屏，`Enter` 键查看下一行，`B` 键查看上一屏。
{% endhighlight %}

## 查看内核参数 sysctl

{% highlight shell %}
$ sysctl -a # 查看 `Linux` 系统内核的全部参数。
{% endhighlight %}

## 显示登陆用户 w

{% highlight shell %}
$ w # 显示当前登陆的用户及其正在执行的命令。
{% endhighlight %}

## 升级系统

{% highlight shell %}
$ do-release-upgrade # 升级。
$ reboot # 升级完毕需要重启。
{% endhighlight %}

## 查看系统版本信息

{% highlight shell %}
$ uname -a # 打印当前系统相关信息，`-a` 表示全部信息。
$ lsb_release -a # 显示 LSB 和版本信息，`-a` 表示全部信息。
$ cat /etc/issue # 查看系统版本信息。
{% endhighlight %}

## 修改文件名 rename

{% highlight shell %}
$ find . -exec rename 's/<from>/<to>/' {} ";" # 批量修改当前目录下的文件名中字符串 <from> 为 <to>。
{% endhighlight %}

## 字符串替换 find | sed

{% highlight shell %}
$ find <filename> -type f -print0 | xargs -0 sed -i 's/<from>/<to>/g' # 把文件 <filename> 中的字符串 <from> 全部改为 <to>。
{% endhighlight %}

## 参考
* [Why Penguin is Linux logo? - LinuxScrew: Linux Blog](http://www.linuxscrew.com/2007/11/14/why-penguin-is-linux-logo)
* [《Linus Torvalds自传》摘录 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2012/09/linus_torvalds.html)
* [Linux命令大全（手册）](http://man.linuxde.net)
* [...](http://github.com/mistydew)
