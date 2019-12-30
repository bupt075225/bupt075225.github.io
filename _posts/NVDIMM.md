NVDIMM



最近公司的虚拟化产品需要支持Intel的非易失性内存设备，了解一下这种设备相关的基本概念和使用方法。

DIMM(Dual Inline Memory Module)，双排直插内存模块，俗称内存条。传统内存的一个特点是掉电数据会丢失。NVDIMM(Non-violate DIMM)，掉电不会丢数据，和固态硬盘(SSD)一样，可以持久化保存数据，同时，读写数据的速度又比硬盘快，理论上可以接近内存的访问速度。NVDIMM使用的是内存总线，扩展性不如SSD，它的应用场景应该不是海量数据存储，而是用于实现高性能。



NVDIMM,PMEM,AEP(Apache Pass)，通常都是指持久化内存。



Region

主板上的DIMM插槽称为socket，在Linux系统中，一个或多个socket上的NVDIMM抽象为一个region，它有PMEM和BLK两种类型，类型决定了以什么样的方式访问持久化存储。

\# ndctl list --dimms
[
  {
    "dev":"nmem1",
    "id":"8089-a2-1904-00003054",
    "handle":272,
    "phys_id":69,
    "flag_failed_flush":true,
    "flag_smart_event":ture
  },
  {
    "dev":"nmem0",
    "id":"8089-a2-1904-0000310e",
    "handle":256,
    "phys_id":66,
    "flag_failed_flush":true,
    "flag_smart_event":ture
  }
]

\# ndctl list --regions
[
  {
    "dev":"region0",
    "size":270582939648,
    "available_size":0,
    "type":"pmem",
    "iset_id":210280806750034028,
    "persistence_domain":"unknown"
  }
]

Namespace

Region划分为一个或多个namespaces，在/dev目录下有/dev/pmem设备与namespace对应，只有划分了namespace后，软件才能访问到持久化内存。Namespace继承region的类型属性，但它还有mode属性，属于同一个region的不同namespace有相同的类型属性，但可以有不同的mode属性。Mode定义namespace的软件特性，包括如下这些：raw, sector, memory, dax。

\# ndctl list --namespaces
[
  {
    "dev":"namespace0.0",
    "mode":"fsdax",
    "map":"dev",
    "size":266352984064,
    "uuid":"1a0a70bd-d397-4261-890b-a37dd9290000",
    "blockdev":"pmem0"
  }
]

BTT(Block Translation Table)

传统硬盘的最小IO单元是扇区(sector)，对一个扇区的一次IO是原子操作，要么成功，要么失败，不可能部分字节成功，这是硬盘硬件实现上支持的。持久化内存却不保证这样的原子操作，因此，Linux系统中的BTT模块实现了软件上的扇区原子操作，实现原理的核心是写重定向(ROW, Redirect On Write)。





<https://nvdimm.wiki.kernel.org/?spm=a2c4e.10696291.0.0.740119a45FiDxh>

<https://www.suse.com/c/nvdimm-enabling-suse-linux-enterprise-12-service-pack-2/?spm=a2c4e.10696291.0.0.69fd19a4Chd8ng>

<https://www.suse.com/c/nvdimm-enabling-part-2-intel/?spm=a2c4e.10696291.0.0.4a3f19a4lAi0Eo>
