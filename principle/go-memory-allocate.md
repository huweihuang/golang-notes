# 1. 内存分配

## 1.1. 内存分配器的问题

当给不同大小的变量分配连续地址的内存的时候，可能因为部分变量内存的回收导致在分配新的内存需求时无法利用被回收的内存地址，因此内存管理不当，容易导致内存的碎片。

## 1.2. 基本策略

1. 每次从操作系统申请⼀⼤块内存（⽐如 1MB），以减少系统调⽤。
2. 将申请到的⼤块内存按照特定⼤⼩预先切分成⼩块，构成链表。
3. 为对象分配内存时，只需从⼤⼩合适的链表提取⼀个⼩块即可。
4. 回收对象内存时，将该⼩块内存重新归还到原链表，以便复⽤。
5. 如闲置内存过多，则尝试归还部分内存给操作系统，降低整体开销

## 1.3. 内存分配的本质

针对不同大小的对象，在不同的 cache 层中，使用不同的内存结构；将从系统中获得的一块连续内存分割成多层次的 cache，以减少锁的使用以提高内存分配效率；申请不同类大小的内存块来减少内存碎片，同时加速内存释放后的垃圾回收。

# 2. Golang内存分配

## 2.1. 内存块

go的内存分配器将内存页分成67个不同大小规格（size class）的块，最小为8KB，最大为32768KB。

内存块的分类：

- span:由多个地址连续的页（page）组成的大块内存。面向内部管理。
- object：将span按照特定的大小切分成多个小块，每个小块可以存储一个对象。面向对象分配。

## 2.2. 内存分配器

内存分配器的三个数据结构（申请逐级向上）：

- mcache：goroutine cache，可以认为是 local cache。不涉及锁竞争。
- mcentral：全局cache，mcache 不够用的时候向 mcentral 申请。涉及锁竞争。
- mheap：当mcentral 也不够用的时候，通过 mheap 向操作系统申请。

## 2.3. 内存分配及回收流程

### 2.3.1. 内存分配的流程

1. object size < 16K，使用 mcache 的小对象分配器 tiny 直接分配。 
2. object size > 32K，则使用 mheap 直接分配。
3. object size > 16K && object size < 32K，先使用 mcache 中对应的 size class 分配。 
4. 如果 mcache 对应的 size class 的 span 已经没有可用的块，则向 mcentral 请求。 
5. 如果 mcentral 也没有可用的块，则向 mheap申请，并切分。 
6. 如果 mheap 也没有合适的 span，则想操作系统申请。

### 2.3.2. 内存回收的流程

- mcache 归还内存分两部分：归还mcentral内存，可能涉及锁竞争；除此之外，归还到mheap，直接插入链表头。
- mcentral 归还mheap。
- mheap 定时归还系统内存。

# 3. tcmalloc(thread-caching mallo)

tcmalloc是google推出的一种内存分配器。

具体策略：全局缓存堆和进程的私有缓存。

1.对于一些小容量的内存申请试用进程的私有缓存，私有缓存不足的时候可以再从全局缓存申请一部分作为私有缓存。

2.对于大容量的内存申请则需要从全局缓存中进行申请。而大小容量的边界就是32k。缓存的组织方式是一个单链表数组，数组的每个元素是一个单链表，链表中的每个元素具有相同的大小。

golang语言中MHeap就是全局缓存堆，MCache作为线程私有缓存。

内存池就是利用MHeap实现，大小切分则是在申请内存的时候就做了，同时MCache分配内存时，可以用MCentral去取对应的sizeClass，多线程管理方面则是通过MCache去实现。



总结:

1.MHeap是一个全局变量，负责向系统申请内存，mallocinit()函数进行初始化。如果分配内存对象大于32K直接向MHeap申请。

2.MCache线程级别管理内存池，关联结构体P，主要是负责线程内部内存申请。

3.MCentral连接MHeap与MCache的，MCache内存不够则向MCentral申请，MCentral不够时向MHeap申请内存。



参考：

- [TCMalloc : Thread-Caching Malloc](http://goog-perftools.sourceforge.net/doc/tcmalloc.html)
- [图解Go语言内存分配](https://mp.weixin.qq.com/s/En1TXJOnAXcGTymZ53zjLg)
- [【golang 源码分析】内存分配与管理](https://blog.csdn.net/zhonglinzhang/article/details/74626412)

