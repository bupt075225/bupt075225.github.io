Keepalived包含VRRP和健康检查两个子系统，VRRP子系统实现VRRP协议(Virtual Router Redundancy Protocol)，健康检查子系统实现对三、四、五层协议及服务状态的检查，例如检查HTTP协议，SMTP，SSL协议，TCP链接以及其它自定义检测项。keepalived有三个进程，主进程和两个子进程，子进程分别实现了VRRP和健康检查子系统，主进程会监控子进程状态。软件架构如下图所示：

![](https://i.imgur.com/LbJMzo2.jpg)

图片来自官方文档

### 线程调度框架(scheduling framework)

Keepalived使用一个多线程调度框架，实现了以下几种类型的线程：

* VRRP包分发线程

接收VRRP通告包，分发到VRRP实例状态机，设置收包超时时间，收包超时就通知VRRP实例状态机。

* 定时器线程

定时执行例测脚本，定时发送免费ARP。

* 事件线程

处理初始化事件，例如：初始化VRRP包分发器，初始化例测脚本。

* 子进程退出状态检查线程

检查子进程退出状态并作相应处理，例如：例测脚本执行完成后，检查执行结果。收到子进程状态变化信号SIGCHLD，会将该进程的退出状态检查线程变为就绪状态。

除事件线程之外，其他线程有两种状态：准备就绪和空闲。

调度优先级：事件线程 > 准备就绪的线程


### I/O复用和包分发器(I/O multiplexer)

运行VRRP协议实例的节点会发送通告包，master节点发送通告包告诉其他节点它的状态，backup节点发送通告包参加选主。一个节点上可以同时运行多个VRRP实例，它们可能是共用一个或多个物理网络接口收发送通告包。VRRP包分发器将所有VRRP实例的读和写两个socket组成一个socket pool，socket与物理网络接口绑定，使用I/O多路复用模型来处理多个socket数据收发。

socket pool中的每个socket都有一个处理线程，即VRRP包分发线程，它会把接收到的数据分发给VRRP实例，然后触发VRRP实例状态机的运行。

此外，进程将收到的信号通过管道发送到I/O多路复用模型，在执行线程调度时以同步调用的方式执行信号处理函数。

线程调度框架和VRRP包分发器的关系如下图所示：

![](https://i.imgur.com/sS0lNVe.jpg)


VRRP子系统实现VRRP协议，IO多路复用模型和事件处理是VRRP子系统的两个关键流程。

* VRRP实例IO多路复用模型

VRRP协议通告，子进程信号状态通过IO多路复用，转为准备就绪的收包事件。

使用sock pool来实现VRRP实例分发器。每一个VRRP实例会对应pool中的一个sock(一对收发socket)，也就是每一个VRRP实例的虚拟网口(对应配置了虚拟IP的网络接口)上绑定了一对收发socket。

sock的收包处理函数是vrrp\_read\_dispatcher\_thread(),它负责收数据包并分发到VRRP实例，然后VRRP实例状态机开始处理。如果虚拟网口故障导致收包超时，VRRP实例状态机开始处理收包超时。

* VRRP事件处理函数launch\_scheduler()

launch\_scheduler()是一个无限循环函数。一次循环取出一个事件，执行对应的事件处理函数。优先处理通用事件(初始事件，发送邮件事件，运行例测脚本)；如果没有通用事件，就处理准备就绪的事件；如果没有准备就绪的事件，就处理收发包事件，并将收发包事件变成准备就绪的事件；最后处理定时器超时事件，将超时的定时器变成准备就绪的事件。以上几类事件遍历完都没找到事件，最后再检查一次准备就绪的事件(收发包事件和定时器超时事件可能变成了就绪事件)。

### VRRP协议状态机

用户配置的每一个VRRP实例都是按照VRRP协议运行，VRRP协议通过状态机实现，收到其它节点发出的通告包(收包事件)和节点维护的定时器(超时事件)推动着状态机的运行。例如，通告发送间隔时间超时，master状态的节点就会发送通告；master\_down\_timer超时，backup状态的节点就会发送通告参加新主选举。VRRP实例有一个时间沙漏，表示超时时间，根据实例的当前状态，会将这个超时时间设置为通告间隔adv\_int或master\_down\_time（约等于3*adv_int,还要加上一个微小抖动）。例如：backup状态的节点设置沙漏为master\_down\_time，实现时刻准备参加新主选举；master状态的节点设置沙漏为adv\_int，实现定时发送通告。

>收包事件，定时器超时事件，故障例测事件推动着状态机的状态迁移。

VRRP协议状态机如下图所示：

![](https://i.imgur.com/rhQfdTi.jpg)


在VRRP协议的实现过程中，定义了如下三类状态：

* state：VRRP实例状态机的状态，包括init,backup,master三个由VRRP协议(RFC2338 6.3)定义的状态和gotomaster,fault两个程序自定义内部状态。

* wantstate：用户在配置文件中配置的状态，包括backup,master两个状态，默认状态是backup，此外它还可能是运行过程中产生的gotomaster和gotofault状态。

* init_state:VRRP实例的初始状态，默认状态是backup，加载配置文件时，修改为用户配置的VRRP实例状态。

状态转移的条件包括：

* master\_down\_timer超时(大约是通告间隔时间的3倍)
* 接收到通告包
* 通告间隔时间超时
* 例测脚本或网络接口检测到被检测对象的状态有变化。
 
前两个条件是通过接收到通告包或接收通告包超时来触发，每次成功接收到通告包或发送通告包后，都会启动定时，设置下次收包或发包的超时时间。第四个条件会在状态处理时进行检查。

如果接收到通告包携带的节点优先级与本地VRRP实例优先级相同时，则继续比较收发通告包的IP地址大小，大的则优先级高。0和255是两个特殊的优先级，0表示master宣布退出集群，需要重新选主，255表示拥有虚拟IP。

* master状态

通告间隔时间adv_int超时，触发master状态处理。

如果没有设置虚拟IP(说明是刚变为master)，配置虚拟IP，路由，规则；立即发送免费ARP；重置免费ARP定时器，启动延迟免费ARP定时器。

如果虚拟IP已经设置，免费ARP定时器已经超时，就发送免费ARP。

发送通告信息。

* master状态收到通告

检查通告包合法性，忽略非法包。

比较IP地址大小。

1）如果通告包中的优先级为0，就发送通告包；

2）如果配置了higher\_prio\_send\_advert选项且通告优先级大于VRRP实例优先级，，就发送通告包；

3）如果通告优先级等于VRRP实例优先级，但通告节点的IP地址大于VRRP实例的IP地址，就发送通告包；

4）如果收到一个低优先级的通告，就发送通告包和免费ARP，启动延迟免费ARP定时器；

5）如果收到优先级相同但通告IP比VRRP实例的IP小，发送通告包和免费ARP，启动延迟免费ARP定时器；

6）如果收到的通告优先级是255，VRRP实例优先级也是255，说明冲突了，就把VRRP实例配置的优先级减1；

7）如果收到一个高优先级的通告，设置master\_down\_time为adv\_int*3，修改wantstate和state为backup，删除虚拟IP及相关网络配置；

8）如果收到优先级相同但通告IP比VRRP实例优先级高，设置master\_down\_time为3个通告周期，修改wantstate和state为backup，删除虚拟IP及相关网络配置；

* backup状态

接收通告包。

检查通告包合法性，忽略非法包，设置master\_down\_time为adv\_int*3。

1) 如果通告包优先级为0(说明master退出集群，不参与选举，请求选举新master)，设置master\_down\_time为SKEW，准备参加新主选举。

2) 如果是非抢占模式，就更新master\_down\_time为adv\_int*3；

3) 如果通告优先级大于等于VRRP实例优先级，就更新master\_down\_time为adv\_int*3

4) 如果配置了抢占延迟选项，而且抢占延迟未超时或延迟时间为0，就更新master\_down\_time为adv\_int*3。

在满足了上述2),3),4)的条件下：

如果配置了抢占延迟，而且通告优先级大于VRRP实例优先级，就把抢占延迟时间设置为0(只能去抢占低优先级的master)；

如果配置了抢占延迟，而且通告优先级小于等于VRRP实例的优先级，就设置抢占延迟超时时间；

在上述1),2),3),4)都不没足的条件下：

就将wantstate改为gotomaster，发送通告，参加新主选举。等到超时adv\_int后，变为master状态并发送通告。

### 检测故障

VRRP通过两方面进行故障检测，一是master周期性发送通告信息，backup在超过大约3个通告周期时间都没收到master发出的通告信息就认为master故障了，开始发出通告信息重新选主。二是例测网络接口状态(UP或DOWN)和检查例测脚本执行结果来判断网络和业务进程是否故障。

配置文件的vrrp\_instance区域中有track\_interface和track\_script配置选项，通过例测网络接口和脚本执行结果来改变优先级权重，从而改变优先级，最终触发主备切换。权重有效取值在-254到254之间。

* 检测网络接口状态

VRRP进程通过netlink从内核中查询到网络接口状态,if\_linkbeat\_refresh\_thread()函数实现例测网络接口状态是UP或DOWN。：

如果跟踪的网口状态是UP且权重大于0，就把它的权重加到全局权重上，如果是DOWN且权重小于0，就从全局权重上减下来。

如果权重被设置为0，只要例测脚本或跟踪的网口任一一个故障，就会进入故障状态。这一点是通过宏VRRP\_ISUP来判断的。

* 例测脚本

vrrp\_script区域与vrrp\_instance区域同级，vrrp\_script区域定义脚本名字、脚本执行间隔和脚本执行结果对应的优先级变更。然后在vrrp\_instance区域里面通过track\_script配置选项来引用，类似于编码中的先定义再引用，如下配置示例：

    vrrp_script check_running {
	    script "/usr/local/bin/check_running"
	    interval 10
	    weight 10
        fall f    # 连续f次失败才认为检测到故障
        rise r    # 连续r次成功才算成功
    }

    vrrp_instance http {
        track_script {
            check_running
        }
    }

vrrp\_script\_thread()是例测脚本定时器的处理函数，它会启动子进程来执行例测脚本。

用户可以配置例测脚本执行成功或失败的次数，即只有连续成功rise次才算检测成功，连续失败fall次才算检测失败,result取值决定例测脚本上一次执行后的检测结果(OK或KO，KO: 0<=result<=rise-1, OK: rise<=rise<=rise+fall-1)。

检测结果KO和OK状态之间的转移过程如下图所示，脚本执行结果(Fail或Success)与上次执行后得到的result值共同决定了如何更新result的值。

![](https://i.imgur.com/cgu6b8e.jpg)


在子进程事件处理函数vrrp\_script\_child\_thread中处理脚本执行结果，更新result值。

    status = WEXITSTATUS(wait_status);
    /* 脚本返回0表示执行成功，非0表示失败 */
	if (status == 0) {
		/* success */
		if (vscript->result < vscript->rise - 1) {
			/* result < rise - 1，表示当前还是KO状态，KO状态下，调用成功则result+1 */
            vscript->result++;
		} else {
            /* 表示当前是KO状态转变成OK状态,或已经处于OK状态，这时直接把result改成最大。要变回KO的话，还需要fall-1次失败 */
			if (vscript->result < vscript->rise)
				log_message(LOG_INFO, "VRRP_Script(%s) succeeded", vscript->sname);
			vscript->result = vscript->rise + vscript->fall - 1;
		}
	} else {
		/* failure */
		if (vscript->result > vscript->rise) {
            /* result > rise，表示当前是OK状态，OK状态下，脚本执行失败的话，result-1 */
			vscript->result--;
		} else {
            /* 表示当前是OK状态转变成KO状态，或已经处于KO状态，这时直接把result改到最小，要变回在OK的话还需要rise次成功 */
			if (vscript->result >= vscript->rise)
				log_message(LOG_INFO, "VRRP_Script(%s) failed", vscript->sname);
			vscript->result = 0;
		}
	}

vrrp\_script\_weight()计算例测脚本的权重，计算方法是：

>脚本的权重值为正时，脚本检测成功的次数result达到rise值，即检测成功，权重值加到优先级上，检测失败时不加；

>主失败：主priority < 从priority + weight，会切换

>主成功：主priority > 从priority + weight，主依然为主

>脚本的权重值为负时，脚本检测成功的次数result小于rise值，即检测失败，就减去权重，检测成功时不影响优先级；

>主失败：主priority - abs(weight) < 从priority，会切换

>主成功：主priority > 从priority，主依然为主

>每个例测脚本均按此方法计算权重，得到的权重结果累加后用来更新优先级。


vrrp\_update\_priority()优先级更新定时器的处理函数，根据例测端口和例测脚本的权重来更新优先级。vrrp\_init\_state()函数中启动优先级更新定时器。


### health-check

根据配置文件产生checkers队列(即需要检查哪几层的服务)，然后将checker注册到I/O调度器上，定时检测。

TCP连接检查函数：tcp\_connect\_thread()，周期性执行检查，发起tcp连接。

tcp\_check\_thread()   写线程的执行函数，发出tcp连接请求后，检测TCP连接状态。

update\_svr\_checker\_state()  每个服务器有一个或多个checker，该函数更新某个checker的检测结果。

perform\_svr\_state() 根据checker检测结果和服务器的当前状态(是否alive)来决定将一台真实服务器加入或移出LVS集群。

HTTP checker的实现过程：

http\_connect\_thread() -> http\_check\_thread() -> http\_request\_thread() -> http\_response\_thread() -> http\_read\_thread() -> http\_handle\_response()

### 数据结构

Keepalived主要数据结构如下图所示：

![](https://i.imgur.com/srmTXn9.png)


vrrp\_data\_t结构体定义了一堆链表。VRRP实例挂到vrrp链表，sock pool中的每个sock挂到vrrp\_socket\_pool链表(形成一个pool),vrrp\_index\_fd指向1024+1个链表头区域，即1025个链表，每个链表是一个槽位，槽位索引号是收包socket描述符模1024再加1，从而形成哈希链表。每个槽位上挂载的是同一个虚拟网口上的所有VRRP实例。例测脚本挂到vrrp\_script链表。

vrrp\_t结构体定义了一个VRRP实例的所有参数。函数alloc\_vrrp()在分配了一个VRRP实例资源并初始化后，会将它挂到vrrp\_data\_t的成员vrrp链表上。

interface\_t结构体定义了配置虚拟IP的网络接口。init\_interface\_queue()->netlink\_interface\_lookup()通过netlink查询网络接口信息，查询到的每个接口信息挂到全局变量if\_queue表示的链表上。

_sock结构体定义了sock pool中的每个sock，其中包括收发两个socket。

thread\_master\_t结构体定义了几种类型的事件链表，收包事件、发包事件、准备就绪的事件、定时器超时事件等。全局变量master维护了这样一个结构体实例。

thread\_t结构体定义了事件，每种类型事件都有一个事件处理函数和事件超时时间。

vrrp\_script_t结构体定义外部脚本，用来跟踪进程状态。







alloc\_vrrp(),分配一个VRRP实例，并初始化。

vrrp\_init\_state(),初始化VRRP实例的状态。

首先将配置文件中配置的实例状态(MASTER或BACKUP)置为VRRP实例的初始状态(wantstate和init_state)，然后在vrrp\_init\_state()中更新实例状态(state)，如果配置的优先级为255(最高优先级)或MASTER，实例状态就更新为GOTO_MASTER，否则更新为BACK。

start\_vrrp\_child()->thread\_make\_master()，创建7个线程链表。launch\_scheduler()循环调度线程。
