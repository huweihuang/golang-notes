> 本文简单介绍下Go垃圾回收三色标记法的步骤。待完善...

# 1. 三色标记法

- 黑色：没有指向（引用）白色集合中的任何对象
- 灰色：可能指向（引用）白色集合中的某些对象
- 白色：剩下的需要被回收的候选对象，当灰色集合为空时，表示白色集合中的对象都没有被引用，那么这些对象就可以被回收。

# 2. 垃圾回收循环的步骤

1. 将所有的对象都放入白色集合中
2. 扫描所有roots对象，放入灰色集合中，roots对象表示在应用中可以被直接访问，一般是全局变量和其他在栈中的对象。
3. 将灰色集合中的某个对象放入黑色集合，然后扫描这个对象有引到到的白色集合中的对象，将那些白色集合中引用到的所有对象放入灰色集合，以此类推，将灰色集合中的对象不断放入黑色集合中，然后白色集合中的对象不断放入灰色集合中。
4. 当灰色集合中的对象为0，即都被放入到黑色集合中了，表示没有任何对象会引用到白色集合中的对象了，因为黑色集合存放不会引用白色集合对象的元素，而灰色集合为0，也不存在引用白色集合对象的元素。所以白色集合中的对象即是没有被引用的对象，可以回收的对象。



参考：

- [Go 1.5 concurrent garbage collector pacing](https://docs.google.com/document/d/1wmjrocXIWTr1JxU-3EQBI6BK6KgtiFArkG47XK73xIQ/edit#heading=h.xy314pvxblbm)
- [Golang’s Real-time GC in Theory and Practice](https://making.pusher.com/golangs-real-time-gc-in-theory-and-practice/)
- [The Journey of Go's Garbage Collector](https://blog.golang.org/ismmkeynote)
- [Golang's Realtime Garbage Collector Video](https://pusher.com/sessions/meetup/the-realtime-guild/golangs-realtime-garbage-collector)
- [How does tricolor garbage collection work](https://jameshfisher.com/2016/11/11/tricolor-gc/)

