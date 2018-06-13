---
title: 一个echo引起的进程崩溃
date: 2016-07-02 16:00:00
categories: PHP
tags: PHP
---
最近在编写后台程序时遇到一个问题。发现后台程序总是莫名其妙的die掉。经排查，发现罪魁祸首是一个echo。
**案情重现：**
1.ssh登陆服务器
2.新建一个php文件test.php，代码如下：

```
<?php
sleep(50);
echo "aaa\n";
file_put_contents("/tmp/test.txt",time());
?>
```

3.用以下命令执行test.php程序
$ php test.php &
查看 /tmp/test.txt 文件的内容为 1381325659
4.然后再次执行如下命令。命令执行后，马上使用exit命令退出登陆
$ php test.php &
5.然后ssh登陆服务器，发现/tmp/test.txt 文件的内容依然是 1381325659。说明第二次执行test.php时，file_put_contents函数没有执行，或者没有执行成功。

**追查真凶**
为什么第二次执行的时候file_put_contents函数没有执行，或者没有执行成功?
让我们使用万能的strace命令查看下程序执行过程中，系统调用情况。
正常情况下，strace结果中会有如下显示：
nanosleep({50, 0}, {50, 0}) = 0
write(1, “aaa\n”, 4aaa
) = 4
lstat(“/tmp/test.txt”, {st_mode=S_IFREG|0644, st_size=10, …}) = 0
lstat(“/tmp”, {st_mode=S_IFDIR|S_ISVTX|0777, st_size=1576960, …}) = 0
open(“/tmp/test.txt”, O_WRONLY|O_CREAT|O_TRUNC, 0666) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=0, …}) = 0
lseek(3, 0, SEEK_CUR) = 0
write(3, “1381326235”, 10) = 10
close(3) = 0
close(2) = 0
close(1) = 0
munmap(0x2b4dd1414000, 4096) = 0
close(0) = 0

如果退出ssh后，再登陆，strace结果中会有如下显示：
write(1, “aaa\n”, 4) = -1 EIO (Input/output error)
close(2) = 0
close(1)
munmap(0x2b4412622000, 4096) = 0
close(0) = 0

注意两次strace的结果蓝色部分。根据strace结果，可以肯定是因为echo时产生了EIO错误导致进程终止，最终没有执行file_put_contents函数。

**一追到底**
为什么退出登陆后，再登陆，就会发生EIO错误呢？这个和linux的会话处理有关。
当用户ssh登陆一个服务器时，也就开始了一个会话。会话开始后，标准输入（stdin）、标准输出（stdout）、标准错误（stderr）会连接到一个对应的终端（pty）。

用户登陆后，任何标准输出都会在终端中有反应。标准输出的文件句柄是1。因此，php中的echo(“aaa\n”) 会导致执行系统调用write(1, “aaa\n”, 4). 会在终端中写aaa\n。

当用户退出登陆时，一个会话就结束了。会话结束时，修改所有打开该终端的文件句柄，改成不可读也不可写；

用户退出登陆后再执行write(1, “aaa\n”, 4)，会报EIO错误。因为终端句柄已经不可写。EIO错误发生后，导致进程结束。

**解决办法**
方法一：
使用重定向符号&把标准输出重定向到空洞。
$ php test.php > /dev/null 2 >&1 &
方法二：
使用nohup。[推荐使用此方法]
$ nohup php test.php &