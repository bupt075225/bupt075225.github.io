Padata是内核态使用多CPU并行处理任务，任务序列化，保证任务提交顺序和执行后返回顺序一致的一种并行计算机制。最初是用于IPsec数据加解密，需要对大量的数据包加解密，同时又不能让这些包被并行加解密后产生乱序。但Padata的实现是一个很通用的机制，不仅仅用在IPsec数据加解密，也可以用于其它需要在内核态做并行任务处理的场景。

### 工作原理

Padata使用内核工作队列机制，工作队列线程执行任务，实现多线程并行任务处理。用户提交的任务请求在Padata内部的处理过程如下图所示：

![](https://i.imgur.com/ImL1K4X.jpg)


所谓的reorder是指，选定一个CPU，从它的并行任务的reorder链表中取出已完成并行处理的工作(一个或多个)，然后加某个CPU的串行任务链表。串行链表对应的CPU由用户指定，当指定的CPU不在Padata可用CPU集合内，会返回错误，为避免返错，用户在调用Padata做并行处理前，可以使用后面要讲到的算法更新用户指定的CPU。

在每个CPU上都有上图中绿色标识的三种链表，并行任务分发到哪个CPU上的任务链表？从哪个CPU的reorder链表中取出任务来准备做串行处理？串行任务链表中的任务放到哪个CPU上执行？都是由下面这个算法来决定的。

    unsigned int cpu_index, cpu, n, i;
    cpumask_var_t cpumask;

    num_cpus = cpumask_weight(cpumask);
    cpu_index = n % num_cpus;
    cpu = cpumask_first(cpumask);
    for (i = 0; i < cpu_index; i++)
        cpu = cpumask_next(cpu, cpumask);

其中，n是并行任务计数或已完成并行处理和reorder处理的任务计数，也可能是CPU ID。

>这个算法保证了依次使用CPU掩码中的CPU来处理任务，n不增加就不会选择新的CPU来执行任务，并行任务的序列化就靠它来实现。

举个例子，Padata使用CPU掩码指定了使用CPU1,CPU3,CPU5,CPU7三个CPU来进行并行和串行处理，随着n的递增，依次选中CPU1,CPU3,CPU5,CPU7,计算数据如下表所示：

    | cpu | n | num_cpus | cpu_index |
    | ----|---|----------|-----------|
    | 1   | 0 | 4        | 0         |
    | 3   | 1 | 4        | 1         |
    | 5   | 2 | 4        | 2         |
    | 7   | 3 | 4        | 3         |
    | 1   | 4 | 4        | 0         |


### 数据结构

Padata有5个核心数据结构：Padata实例结构体，Padata内部全局控制结构体，并行处理的工作结构体，串行处理的工作结构体，嵌入到用户任务请求中的数据结构体。

![](https://i.imgur.com/0b5CpnD.png)




### 使用Padata

* 步骤1 — 创建padata实例

实例化padata_instance结构体。

    #include <linux/padata.h>
    struct padata_instance *padata_alloc(struct workqueue_struct *wq,
                                      const struct cpumask *pcpumask,
                                      const struct cpumask *cbcpumask);

pcupmask指定哪些CPU用来执行该实例的并行任务，cbcpumask指定哪些CPU用来执行该实例的串行回调任务，wq是用来执行任务的多线程工作队列。

该接口函数同时还会初始化Padata的内部控制结构体parallel_data。

* 步骤2 — 启动Padata

使用下面两个函数分别启动和关闭Padata实例：

    int padata_start(struct padata_instance *pinst);
    void padata_stop(struct padata_instance *pinst);

它们设置或清除实例状态标志PADATA_INIT，如果该标志没有被置上，Padata的接口函数不会被执行。

* 步骤3 — 提交任务到Padata实例

在提交任务到Padata之前，要实例化padata_priv结构体，提供并行处理函数parallel()和串行处理函数serial()。

任务提交接口函数：

    int padata_do_parallel(struct padata_instance *pinst,
                           struct padata_priv *padata, int cb_cpu);

pinst和padata在创建Padata实例那一步已经设置好，当任务执行完成后，cb_cpu指定由哪个CPU来执行最后的回调。

调用成功返回0，如果返回-EBUSY表示正在修改实例的CPU掩码，返回-EINVAL表示cb_cpu不在实例可用的CPU集合内，或者实例状态标志没有被置。

成功提交任务后，Padata会调用padata_priv结构体提供的并行处理函数parallel()。是通过工作队列线程的执行函数来调用parallel()函数，线程执行函数会将软中断被禁掉，所以parallel()函数不能睡眠，此外，也不要在parallel()中做同步等待或耗时操作，应该尽快返回。

padata\_priv结构体是parallel()函数的唯一入参，因为padata\_priv结构体是嵌入到用户数据结构体中的，所以可通过container_of()获取到用户数据结构体指针，它会提供处理任务所需的信息。

parallel()函数没有返回值，Padata认为一旦parallel()开始执行后，就由parallel()负责任务,Padata就只能送它到这里了。

* 步骤4 — 通知Padata任务处理完成

任务执行完成后，parallel()函数要通知Padata，如果任务最终由其它函数(非parallel())执行完成，例如异步执行的回调函数，也要通知Padata，一个原则是谁最终执行完成就由谁来通知Padata。

通知Padata任务完成的函数接口是：

    void padata_do_serial(struct padata_priv *padata);

这个函数同时会触发串行处理函数serial()的执行，serial()也是提交到工作队列线程中被执行，与parallel()类似,serial()的执行可能会稍微延迟一下，因为Padata需要执行reorder操作，来保证任务完成的顺序与提交任务的顺序一致，即实现并行任务的序列化。

* 步骤5 — 清理Padata实例

当Padata实例不再使用时，需要调用下面的接口进行清理：

    void padata_free(struct padata_instance *pinst);

该接口会忙等，直到所有任务完成 。