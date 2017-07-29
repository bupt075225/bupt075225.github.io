Kdump是Linux系统的一个功能，当内核崩溃时它会生成core dump文件和内核日志文件，是定位内核崩溃原因的重要手段。在多数Linux发行版本中都默认安装Kdump，开箱即用，但在一些裁剪过的嵌入式Linux系统中并没有包含Kdump。这篇文章简要介绍下移值Kdump的方法。

### Kdump的实现原理

实现Kdump机制的核心是kexec，早在2005年以前，Linux内核开发者开发出了kexec，最初的目的是用来便于调试内核，能够从一个内核直接启动另一个内核，节省等待系统重启的时间。Kdump的实现原理就是在系统内核(the first kernel)崩溃后，启动另一个捕捉内核(the second kernel/kdump kernel)来收集已崩溃内核的内存信息，然后将这些信息打包成core dump文件转储到非易失性存储设备供后续问题分析，最后重启系统。

### 移植Kdump

以下介绍的移植过程是基于Linux 2.6.32版本的内核，x86_64架构。

移植Kdump的过程包括这几部分工作：

- 配置内核编译选项以支持Kdump并重新编译内核；

- 在Grub中设置crashkernel内核启动选项；

- 编译kexec；

- 在系统硬盘分区挂载成功后使用kexec加载捕捉内核(kdump kernel)或转储core dump文件。

步骤1 —— 配置内核编译选项

系统内核与捕捉内核可以是同一个内核，也可以是分别编译的两个内核，最好是使用同一个内核。

执行`make menuconfig`命令，配置以下这些内核选项：

Processor type and features -> kexec system call

Processor type and features -> kernel crash dumps

Processor type and features -> Build a relocatable kernel

Processor type and features -> Physical address where the kernel is loaded

上面这个选项为捕捉内核设置一个加载起始地址，到底应该从什么地址开始？没有一个固定值，如果设置不正确，加载内核时会报错提示加载地址非法，不能访问。通常0x1000000/16M或0x2000000/32M是可以的。可以使用`cat /proc/iomem`命令查看当前系统内核占用的内存之后那片预留内存在什么地址范围，系统内核通常会使用这片预留内存来加载捕捉内核。

在系统Grub的内核启动选项中加上“crashkernel=X@y”这个选项，其中X表示系统内核需要预留的内存大小，用来加载捕捉内核，Y表示预留内存的起始物理地址，就是上面内核选项设置的加载起始地址。例如"crashkernel=128M@32M"表示在32M的起始地址之后预留128M内存。

Kernel hacking -> Compile the kernel with debug info

下面这几个选项如果存在也要打开：

Filesystems -> Pseudo filesystems -> /proc/vmcore support

Filesystem -> Pseudo filesystems -> sysfs file system support

Processor type and features -> 启用高端内存的支持

Processor type and features中支持SMP的选项如果被打开，在kexec命令加载捕捉内核时指定“maxcpus=1”或“nr_cpus=1”选项，捕捉内核就不关闭多CPU支持，如果想要提高dump工具makedumpfile的性能，也可以打开多CPU支持，有兴趣或需要可以进一步研究支持多CPU的方法。

此外，建议将选项打开，可以使用下面的命令触发内核崩溃：

    [root@localhost ~]# echo c > /proc/sysrq-trigger

完成上述内核编译选项配置后，执行`make`命令重新编译内核，使用新内核启动系统。

步骤2 —— 编译kexec

从linux kernel官方站点[下载kexec](https://www.kernel.org/pub/linux/utils/kernel/kexec/)，建议下载版本与内核版本发布时间大致对应，否则会遇到依赖软件的版本不匹配的编译问题。

编译kexec源码后，得到kdump和kexec：

    [root@localhost ~]# tar -xzvf kexec-tools-2.0.0.tar.gz
    [root@localhost ~]# cd kexec-tools-2.0.0
    [root@localhost kexec-tools-2.0.0]# ./configure
    [root@localhost kexec-tools-2.0.0]# make all
    [root@localhost kexec-tools-2.0.0]# ls build/sbin/
    kdump  kexec

将kdump和kexec安装到系统根文件系统`/sbin`目录下。

步骤3 —— 启动Kdump服务

在系统挂载根文件系统成功后，在/etc/inittab文件中找一个加载点来启动Kdump服务。服务启动脚本的主要逻辑是：判断如果存在`/proc/vmcore`，则转储core dump文件到dump目录，否则就使用kexec命令来加载捕捉内核。下面是一个Kdump服务启动脚本的实现：

### 测试Kdump

触发内核崩溃，等系统内核再次启动后，查看dump目录下是否保存了上次的core dump相关文件。

### 后记

以上简要介绍了移植Kdump的过程，重点描述的是整个操作过程，实际操作中并不会很顺利，由于移值的系统环境差异，一定会遇到一些问题，但移植涉及的内容和步骤都上述内容，遇到具体问题只能具体分析，一切真相都在源代码中。在[移植Linux Kdump(二)]()中会介绍Linux系统启动过程涉及的一些重要概念，了解了重要的基本概念对于解决问题会很大帮助。

### 参考链接

1.[深入探索 Kdump](https://www.ibm.com/developerworks/cn/linux/l-cn-kdump1/index.html)

2.[使用 Crash 工具分析 Linux dump 文件](https://www.ibm.com/developerworks/cn/linux/l-cn-dumpanalyse/index.html)

3.[The kexec-based Crash Dumping Solution](https://www.kernel.org/doc/Documentation/kdump/kdump.txt)