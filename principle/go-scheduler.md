> 本文主要介绍Go的调度模型。

# 1. 线程实现模型

线程模型有三类：[内核级线程模型、用户级线程模型、混合型线程模型](https://en.wikipedia.org/wiki/Thread_%28computing%29#Models)。三者的区别主要在于线程与内核调度实体KSE(Kernel Scheduling Entity)之间的对应关系上。

> 内核调度实体KSE指操作系统内核调度器调度的对象实体，是内核调度的最小单元。

## 1.1. 线程模型对比

| 线程模型       | 用户线程与KSE之间的对应关系 | 特点                                                         | 优点                                                         | 缺点                                                         |
| -------------- | --------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 内核级线程模型 | 1:1                         | 1条用户线程对应一条内核进程/线程来调度，即以核心态线程实现。 | 具有和内核线程一致的优点，不同用户线程之间不会互相影响。可以利用多核系统的优势。 | 在大量线程的情况下，线程的创建、删除、切换的代价更昂贵，影响性能。 |
| 用户级线程模型 | M:1                         | N条用户线程只由一条内核进程/线程调度，即以用户态线程实现。   | 线程的创建、删除和环境切换都很高效。                         | 一旦一个线程发生阻塞，整个进程下的其他线程也会被阻塞。不能利用多核系统的优势。 |
| 混合型线程模型 | M:N                         | M条用户线程由N条内核线程动态关联。又称两级线程模型           | 可以快速地执行上下文切换，而且可以利用多核的优势。当某个线程发生阻塞可以调度出CPU关联到可以执行的线程上。目前Go就是采用这种线程模型。 | 动态关联机制实现复杂，需要用户或runtime自己去实现。          |

## 1.2. 线程模型示意图

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1557130409/article/golang/scheduler/thread-model.png" width="100%">

# 2. G-P-M调度模型

**调度模型:**

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1557129220/article/golang/scheduler/go-scheduler.png" width="80%">

**G-P-M对应关系：**

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1557129219/article/golang/scheduler/gpm.png" width="40%">

## 2.1. 基本概念

- M：machine，代表系统内核进程，用来执行G。（工人）
- P：processor，代表调度执行的上下文（context），维护了一个本地的goroutine的队列。（小推车）
- G：goroutine，代表goroutine，即执行的goroutine的数据结构及栈等。（砖头）

## 2.2. 基本流程

调度的本质是将G尽量均匀合理地安排给M来执行，其中P的作用就是来实现合理安排逻辑。

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1557134526/article/golang/scheduler/in-motion.jpg" width="50%">

- P的数量通过 `GOMAXPROCS()` 来设置，一般等于CPU的核数，对于一次代码执行设置好一般不会变。
- P维护了一个本地的G队列（runqueue），包括正在执行和待执行的G，尽量保证所有的P都匹配一个M同时在执行G。
- 当P本地goroutine队列消费完，会从全局的goroutine队列（global runqueue）中拿goroutine到本地队列。P也会定期检查全局的goroutine队列，避免存在全局的goroutine没有被执行而"饿死"的现象。
- P和M是动态形式的一对一的关系，P和G是动态形式的一对多的关系。

## 2.3. 抢占式调度（阻塞）

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1557134526/article/golang/scheduler/syscall.jpg">

当goroutine发生阻塞的时候，可以通过P将剩余的G切换给新的M来执行，而不会导致剩余的G无法执行，如果没有M则创建M来匹配P。

当阻塞的goroutine返回后，进程会尝试获取一个上下文（Context）来执行这个goroutine。一般是先从其他进程中"偷取"一个Context，如果"偷取"不成功，则将goroutine放入全局的goroutine中。

## 2.4. 偷任务

<img src="https://res.cloudinary.com/dqxtn0ick/image/upload/v1557134526/article/golang/scheduler/steal.jpg">

P可以偷任务即goroutine，当某个P的本地G执行完，且全局没有G需要执行的时候，P可以去偷别的P还没有执行完的一半的G来给M执行，提高了G的执行效率。



参考：

- http://morsmachine.dk/go-scheduler
- [Scalable Go Scheduler Design Doc](<https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit#heading=h.mmq8lm48qfcw>)
- [Go Preemptive Scheduler Design Doc](<https://docs.google.com/document/d/1ETuA2IOmnaQ4j81AtTGT40Y4_Jr6_IDASEKg0t0dBR8/edit#heading=h.3pilqarbrc9h>)
- [k2huang/blogpost/Go并发机制](<https://github.com/k2huang/blogpost/blob/master/golang/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/%E5%B9%B6%E5%8F%91%E6%9C%BA%E5%88%B6/Go%E5%B9%B6%E5%8F%91%E6%9C%BA%E5%88%B6.md>)
