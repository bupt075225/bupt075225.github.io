在[移植Linux Kdump(一)]()中介绍了移植Kdump的过程和具体步骤，Kdump的核心思想是在一个内核中启动另外一个内核，如果不了解Linux的启动过程，不利于解决移植过程中出现的问题，即使顺利的移植成功，也只是按图索骥，浮于表面。这篇文章主要介绍Linux系统启动过程中的一些重要基本概念。

从系统启动盘的主引导扇区中读取引导程序(Bootloader/Grub)并加载到内存，Grub从硬盘中将内核映像文件和根文件系统加载到内存，Grub交控制权交过内核，开始内核初始化，然后挂载根文件系统，启动初始进程init。

### 概述

* rootfs

bootloader在将控制权和initrd文件或initramfs交给内核后，由内核负责继续完成系统启动，initrd中的根文件系统包含启动所需要的驱动，内核可以访问根文件系统并加载驱动。在启动过程的后期，内核可以挂载真正的系统根设备来替换initrd，有的嵌入式系统的系统根设备就是承载根文件系统的存储空间。有两种存放根文件系统的空间：ramdisk和缓存。

* ramdisk

将一部分大小固定的内存空间用作块设备，可用来存放根文件系统。典型的应用是initrd。

* ramfs/tmpfs

使用缓存来存放的文件系统，与普通缓存不同点是不能释放用作ramfs/tmpfs的内存空间。ramfs没有大小限制，可以写到内存耗尽；tmpfs可设置占用空间的大小。典型的应用是initramfs。

* initrd

initrd(initial ram disk)是一个ramdisk，上面存放了内核初始化所需要的文件，通常在说initrd文件时指代的是根文件系统压缩包，在bootloader/grub中会配置与内核对应的initrd文件。initrd文件通常是gz格式压缩包。

* initramfs

initramfs(initial ram file system)是被编译链接在内核中的一个文件系统，它本质是一个ramfs或tmpfs。

支持initramfs和initrd的配置选项：

CONFIG_BLK_DEV_INITRD=y

General setup -> Initial RAM filesystem and RAM disk (initramfs/initrd) support

支持ram disk的配置选项：

CONFIG_BLK_DEV_RAM=y

Device Drivers -> Block devices -> RAM block device support


* vmlinux,vmlinux.bin,vmlinuz,bzImage,zImage

vmlinux 是内核源码编译得到的原始文件，是ELF(可执行可链接文件)格式。

vmlinux.bin 是vmlinux的二进制格式文件，不包括符号和可重定位信息。

vmlinuz是vmlinux经gzip压缩后的文件。`make zImage`和`make bzImage`分别创建出zImage和bzImage(big zImage)，它们的头部内嵌了gzip的解压缩代码,zImage解压到低端内存，bzImage解压到高端内存。



initramfs和initrd文件如何构造？

bootloader如何找到initramfs和initrd文件？

kernel挂载根文件系统的过程

/proc/kmsg,内核日志记录



### 参考链接

1.[ramfs, rootfs and initramfs](https://www.kernel.org/doc/Documentation/filesystems/ramfs-rootfs-initramfs.txt)

2.["initrd" and "initramfs"--What's Up With That?](https://www.linux.com/learn/kernel-newbie-corner-initrd-and-initramfs-whats)

3.["initrd" and "initramfs"--Some Unfinished Business](https://www.linux.com/learn/kernel-newbie-corner-initrd-and-initramfs-some-unfinished-business)

