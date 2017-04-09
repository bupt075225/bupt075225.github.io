kickstart是Red Hat系Linux发行版本上用来实现无人值守自动安装和配置操作系统的方法。说直白一点就是，使用安装光盘装Linux系统时，跟着安装向导，设置时区和时间，选择安装语言，硬盘分区等一系列步骤，可以不用手动一步步点选，让这个过程自动完成，安装完成后还能自动配置系统，比如安装常用的第三方软件，数据库建表等，这样节省下的时间可以用来干点更有趣的事，等你回来就是一个可以直接使用的系统。

想让机器自动化，首先得告诉它怎样干，步骤是什么，教会以后它就默默的按指示干活。kickstart配置文件就是给机器的干活说明书，你的指示全写在里面。接下来讲讲如何修改配置文件来实现你的私人定制。


>kickstart有图像化配置工具，让你就像点菜一样，点好你需要的然后就生成文本配置文件，无需了解配置文件格式和语法。如果你喜欢图像化配置，后面的内容就可以不看了。

>cobbler是在kickstart基础上提供更丰富功能的安装配置工具，特别是支持网络引导安装。

建议不要从零开始写配置文件，而是在已有的配置文件基础上修改，最后成为满足需求的配置。这里以CentOS 7为例，安装成功后的CentOS系统都会产生一个kickstart配置/root/anaconda-ks.cfg，复制一份，用vim打开开始编辑：

>vim anaconda-ks.cfg

配置文件主要包含如下几大部分：

- 指令
- %packages ... %end
- %pre ... %end
- %post ... %end

这几部分的顺序不能变，%post和%pre两者必须放最后，而且是可选部分，非必需。以`#`起始的一行是注释。

### 指令部分

介绍一些重要的指令，查询所有选项介绍请参考[KICKSTART OPTIONS](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/Installation_Guide/s1-kickstart2-options.html)

- graphical

图形化安装模式，默认安装模式。与之对应的是文本安装模式。

- text

文本安装模式。

- reboot

安装完成后自动重启。

- selinux

设置系统SELinux的状态，该指令有三个选项：--enforcing,--disable,--permissive,默认选项是--enforcing。

- clearpart

删除分区，要在创建分区之前使用这个指令。

--all　　　　删除所有分区

--drives　　指定删除哪块硬盘上的分区

--initlabel 

- autopart

自动分区

- part

创建一个分区。

### %packages

%packages指令列出需要安装哪些RPM包。包可以根据分组(group)或包名来指定。安装程序定义了一些包含相关包的多个分组，分组清单定义在安装光盘`repodata/comps-*.xml`文件中，每一组都有一个组名，ID，包含的包列表。例如下面是一个指定包分组的例子：

	%packages
	@core	
    %end

示例中指定要分组，一行一个，以@开始，紧跟着组名或组ID。

用包名指定要安装的包，每行一组，可以使用`*`来配置多个包，如：

	%packages
	net-tools
	vim*
	mariadb-server
	mariadb
    %end

在包名或分组名的前面加上`-`表示不安装，例如：

	%packages
	-NetworkManager
	-autofs
    %end

### %post

在系统安装完成后，%post指令提供了一个机会来运行脚本完成额外的一些任务，例如安装额外的非RPM软件或第三方软件，配置系统等。

%post指令的选项：

- --nochroot

在chroot环境之外运行命令，安装过程中根目录是`/mnt/sysimage`，而非常用的`/`，所以chroot环境下，不用在路径前带`/mnt/sysimage`前缀，chroot环境的存在是因为安装RPM包能成功，否则RPM包的安装路径是以常用的`/`为根起点，如果以`/mnt/sysimage`为根起点是会找不到路径的。

- --interpreter /usr/bin/python

指定除shell以外的脚本语言，例如Python。

- --log /path/to/logfile

记录post-intall脚本运行日志。

>注意要考虑是否使用了--nochroot选项，相应的日志文件路径是不同的。例如，使用--nochroot，路径是/mnt/sysimage/root/ks-post.log

### 参考链接

[CREATING THE KICKSTART FILE](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/Installation_Guide/s1-kickstart2-file.html)