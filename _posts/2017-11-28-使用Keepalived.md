[Keepalived](http://www.keepalived.org/)是一个用C语言编写的负载均衡器。该项目的主要目标是为Linux系统和基于Linux的基础设施提供简单而强大的负载均衡和高可用性设施，它与[LVS](http://www.linuxvirtualserver.org/zh/lvs1.html)项目一起可实现一个高可用、高可靠、高性能、可伸缩的Linux服务器集群。Keepalived当然也可以单独应用到其它应用场景，例如，利用它的高可用性实现双机热备。

Keepalived包括两大功能特性：一是使用VRRP协议实现负载均衡器的故障转移，避免出现单点故障，实现高可用性；二是检查服务节点的健康状况，自适应维护和管理服务节点池。后者需要与LVS配合使用。这篇文章主要介绍前一个特性的配置，用来实现故障转移，避免出现负载均衡器单点故障。

## 安装

从[官网](http://www.keepalived.org/)下载软件源码包。解压、编译、安装。

    [root@localhost home]# tar -xzvf keepalived-1.3.5.tar.gz 
    [root@localhost home]# cd keepalived-1.3.5/
    [root@localhost keepalived-1.3.5]# ./configure 
    [root@localhost keepalived-1.3.5]# make;make install

安装完成后，使用`whereis`命令查看安装目录： 

    [root@localhost keepalived-1.3.5]# whereis keepalived
    keepalived: /usr/local/sbin/keepalived /usr/local/etc/keepalived

在/usr/local/etc/keepalived/目录下有keepalived.conf这个文件，它是一个配置样例。

默认情况下，keepalived配置文件放置在/etc/keepalived/目录下，初始安装成功后，/etc/目录下是没有keepalived/目录和配置文件keepalived.conf，需要手动创建该配置文件。

在使用"keepalived -D"命令启动keepalived时，如果没有配置文件会启动失败，日志文件/var/log/messages中有错误提示：

>Unable to find configuration file /etc/keepalived/keepalived.conf (glob returned 3)

执行命令启动keepalived

    [root@westone ~]# keepalived -D

查看日志文件：

    Jun 23 15:29:33 localhost Keepalived[9525]: Starting Keepalived v1.3.5 (03/19,2017), git commit v1.3.5-6-g6fa32f2
    Jun 23 15:29:33 localhost Keepalived[9525]: Unable to resolve default script username 'keepalived_script' - ignoring
    Jun 23 15:29:33 localhost Keepalived[9525]: Opening file '/etc/keepalived/keepalived.conf'.
    Jun 23 15:29:33 localhost Keepalived[9526]: Starting Healthcheck child process, pid=9527
    Jun 23 15:29:33 localhost Keepalived[9526]: Starting VRRP child process, pid=9528
    Jun 23 15:29:33 localhost Keepalived_vrrp[9528]: Registering Kernel netlink reflector
    Jun 23 15:29:33 localhost Keepalived_vrrp[9528]: Registering Kernel netlink command channel
    Jun 23 15:29:33 localhost Keepalived_vrrp[9528]: Registering gratuitous ARP shared channel
    Jun 23 15:29:33 localhost Keepalived_vrrp[9528]: Opening file '/etc/keepalived/keepalived.conf'.
    Jun 23 15:29:33 localhost Keepalived_vrrp[9528]: Using LinkWatch kernel netlink reflector...
    Jun 23 15:29:33 localhost kernel: IPVS: Registered protocols (TCP, UDP, SCTP, AH, ESP)
    Jun 23 15:29:33 localhost kernel: IPVS: Connection hash table configured (size=4096, memory=64Kbytes)
    Jun 23 15:29:33 localhost kernel: IPVS: Creating netns size=2040 id=0
    Jun 23 15:29:33 localhost kernel: IPVS: ipvs loaded.
    Jun 23 15:29:33 localhost Keepalived_healthcheckers[9527]: Opening file '/etc/keepalived/keepalived.conf'.

查看进程

    [root@westone ~]# ps -aux | grep keepalived
    root       3394  0.0  0.0  47800  1044 ?        Ss   11:28   0:00 keepalived -D
    root       3395  0.0  0.0  49872  2160 ?        S    11:28   0:00 keepalived -D
    root       3396  0.0  0.0  49872  1544 ?        S    11:28   0:00 keepalived -D

正常运行时共启动了3个进程，一个是父进程，负责监控子进程，另外两个分别是vrrp子进程和checkers子进程。

## 配置文件

使用两个服务器LB_1和LB_2组成双机热备，组网示意图如下：

![](https://i.imgur.com/fQSIdyv.jpg)

配置文件keepalived.conf在/etc/keepalived目录下，主备节点配置分别如下：

主节点配置：

    ! Configuration File for keepalived

    # 全局定义块
    global_defs {
        router_id node_A           #本节点的标识符,发邮件时才会用
    }

    # VRRP实例定义块
    vrrp_instance VI_1 {
        state MASTER               #主节点
        interface enp3s0f0         #vrrp用于发送组播包的接口,通告主节点状态
        #设置需要监控的网络接口,任意一个出现故障,节点就进入故障状态
        track_interface {
            enp3s0f0
        }
        virtual_router_id 51       #vrrp内唯一的虚拟路由器标识,主备必须一样
        priority 51               #优先级,值越大优先级越高
        advert_int 3               #主向备组播健康状态的间隔,单位为秒
        authentication {
            auth_type PASS         #认证类型,主备必须一样
            auth_pass 1111         #认证密码
        }
        virtual_ipaddress {
            10.10.10.114           #虚拟IP地址列表
        }
        # 当前节点转为主节点后触发的脚本
        notify_master /home/lc/test/change_to_master.sh
        # 当前节点转为备节点后触发的脚本
        notify_backup /home/lc/test/change_to_backup.sh
        # 当前节点出现故障后触发的脚本
        notify_fault  /home/lc/test/change_to_fault.sh
    }

备节点配置：

    ! Configuration File for keepalived

    # 全局定义块
    global_defs {
        router_id node_B           #本节点的标识符,发邮件时才会用
    }

    # VRRP实例定义块
    vrrp_instance VI_1 {
        state BACKUP               #备节点
        interface enp3s0f0         #vrrp用于发送组播包的接口,通告主节点状态
        #设置需要监控的网络接口,任意一个出现故障,节点就进入故障状态
        track_interface {
            enp3s0f0
        }
        virtual_router_id 51       #虚拟路由器ID,主备必须一样
        priority 50                #优先级,值越大优先级越高
        advert_int 3               #主向备组播健康状态的间隔,单位为秒
        authentication {
            auth_type PASS         #认证类型,主备必须一样
            auth_pass 1111         #认证密码
        }
        virtual_ipaddress {
            10.10.10.114           #虚拟IP地址列表
        }
        # 当前节点转为主节点后触发的脚本
        notify_master /home/lc/test/change_to_master.sh
        # 当前节点转为备节点后触发的脚本
        notify_backup /home/lc/test/change_to_backup.sh
        # 当前节点出现故障后触发的脚本
        notify_fault  /home/lc/test/change_to_fault.sh
    }


## 测试主备切换

查看主节点网络接口的IP信息，虚拟IP被绑定在配置文件中interface配置的接口上。

    [root@localhost test]# ip a
    ...
    3: enp3s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
        link/ether 14:18:77:69:82:b9 brd ff:ff:ff:ff:ff:ff
        inet 10.10.10.114/24 brd 10.10.10.255 scope global enp3s0f0
           valid_lft forever preferred_lft forever
        inet 10.10.10.117/32 scope global enp3s0f0
           valid_lft forever preferred_lft forever
        inet6 fe80::1618:77ff:fe69:82b9/64 scope link 
           valid_lft forever preferred_lft forever
    ...

查看备节点网络接口的IP信息，没有绑定虚拟IP

    [root@westone test]# ip a
    ...
    2: enp3s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
        link/ether 0c:c4:7a:db:8f:c8 brd ff:ff:ff:ff:ff:ff
        inet 10.10.10.115/24 brd 10.10.10.255 scope global enp3s0f0
           valid_lft forever preferred_lft forever
        inet6 fe80::ec4:7aff:fedb:8fc8/64 scope link 
           valid_lft forever preferred_lft forever
    ...


主节点停掉keepalived以模拟主节点故障，主节点日志显示如下：

    Jun 27 10:27:21 localhost systemd: Stopped LVS and VRRP High Availability Monitor.

查看备节点日志，可以看到备节点成为主

    Jun 27 10:27:23 localhost Keepalived_vrrp[3416]: VRRP_Instance(VI_1) Transition to MASTER STATE
    Jun 27 10:27:26 localhost Keepalived_vrrp[3416]: VRRP_Instance(VI_1) Entering MASTER STATE
    Jun 27 10:27:26 localhost Keepalived_vrrp[3416]: VRRP_Instance(VI_1) setting protocol VIPs.

原主节点恢复后，根据优先级进行重新选主，进行了主备切换

    Jun 27 10:29:50 localhost Keepalived_vrrp[3416]: VRRP_Instance(VI_1) Received advert with higher priority 120, ours 50
    Jun 27 10:29:50 localhost Keepalived_vrrp[3416]: VRRP_Instance(VI_1) Entering BACKUP STATE

新主节点上，成为主节点后触发脚本输出：

    [root@localhost test]# cat /output.txt 
    Tue Jun 27 10:29:51 CST 2017
    10.10.10.114 become master

## FQA

1. 两个节点组成的集群，配置完keepalived后出现双主

查看防火墙规则：

    [root@localhost ~]# iptables -L

如果设置了规则导致不能收到组播包，清除规则：

    [root@localhost ~]# iptables -F

2.安装配置脚本出现的报错

出现如下错误提示：

>configure: error: libnfnetlink headers missing

缺少netlink开发头文件，需要安装libnfnetlink-devel RPM包：

    [root@localhost keepalived-1.3.5]# rpm -ivh libnfnetlink-devel-1.0.1-4.el7.x86_64.rpm

>*** WARNING - this build will not support IPVS with IPv6. Please install libnl/libnl-3 dev libraries to support IPv6 with IPVS

缺少netlink相关的RPM包，解决方法如下：

    [root@localhost keepalived-1.3.5]# yum install libnl-devel-1.1.4-3.el7.x86_64

3.启动keepalived守护进程失败

在使用systemctl命令来启动keepalived守护进程时，可能会启动失败，提示如下错误：

>PID file /usr/local/var/run/keepalived.pid not readable (yet?) after start.

原因是在systemctl命令的unit文件/usr/lib/systemd/system/keepalived.service中，配置PID文件的位置不正确，通常将PID文件放在/var/run/目录下。修改后的unit文件中应该包含这样一行：

>PIDFile=/var/run/keepalived.pid