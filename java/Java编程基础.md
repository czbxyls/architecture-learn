# Java编程基础


## [并发](http://www.importnew.com/18459.html)

### [Concurrency](http://www.importnew.com/26461.html)

- [ConcurrentLinkedQueue](https://blog.csdn.net/chenguibao/article/details/51773322)

- [ConcurrentHashMap](https://github.com/crossoverJie/JCSprout/blob/master/MD/ConcurrentHashMap.md)

	- JDK1.7

		- 分段锁

			- Segment

				- HashEntry

		- 一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment

		- 数组+链表

			- 数据插入之前检查下是否需要扩容

		- Size方法

			- 在统计的过程插入新的元素

				- 统计过程累计segment的size和modsize

				- 再次累计segment的modsize，如果不等，则重新统计，相等则退出

				- 超过阈值，则对每一个Segment加锁，再重新统计

				- 释放锁，统计结束

	- JDK1.8

		- CAS + synchronized 

		- 差异

			- volatile val和next

			- HashEntry 改为 Node

			- 抛弃了分段锁

		- put操作

			- 根据 key 计算出 hashcode 

			- 判断是否需要进行初始化

			- f 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则通过自旋保证成功

			- 如果当前位置的 hashcode == MOVED == -1,则需要进行扩容

			- 如果都不满足，则利用 synchronized 锁写入数据

			- 如果Node数量大于 TREEIFY_THRESHOLD 则要转换为红黑树

- CopyOnWriteList

### 线程间协作

- wait/notify/notifyAll

- Join

	- 主线程等待子线程执行完毕

- [Condition](https://www.cnblogs.com/dolphin0520/p/3920385.html)

	- 生产者和消费者

- CountDownLaunch

### ThreadLocal

- 为每个使用该变量的线程都提供一个变量值的副本

- [ThreadLocalMap的Key值为弱引用，即WeakReference<ThreadLocal<?>>](https://majiaji.coding.me/2017/03/27/threadLocal-WeakReference%E5%92%8C%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E7%9A%84%E6%80%9D%E8%80%83/)

	- 弱引用，即弱引用引用的对象只被弱引用引用到时，GC扫描时就可以被回收

	- 内存泄露的原因

		- ThreadLocal在局部使用时为强引用，使用set设置value变量，这时候没有调用remove，等到函数退出后为强引用消失

		- ThreadLocalMap的Key在之后由于为弱引用，下一次的GC会自动把ThreadLocal回收

		- 这时候Value值如果没有访问路径关联，即没有使用过（没有再调用get/set/remove），则无法被回收，造成内存泄露

	- 为什么要使用弱引用

		- 弱引用可以首先由gc来判断ThreadLocal实例是否真的可以回收，由gc回收的结果，间接告诉我们，key为null了，这时候value也可以被清理了

		- 如果Key为强引用，则每次退出的时候代码逻辑要显示调用清理方法，Key（即ThreadLocal）才能被回收

	- set/get/remove方法（这些方法会对key为null的entry进行释放）

	- 工程实践

		- 在一个上下文类中声明一个静态的ThreadLocal对象，延长其生命周期，方便其他方法访问

		- 以这个对象为载体存储请求的上下文信息，在各个处理函数中进行传递

		- 最后退出的时候调用remove方法，避免内存泄露

- 可能引发内存泄露

	- 线程内不使用ThreadLocal时，及时使用remove移除

### 锁

- Lock

	- [ReenTrantLock](https://www.cnblogs.com/baizhanshi/p/7211802.html)

	- [与synchronized的区别和联系](https://blog.csdn.net/justloveyou_/article/details/54972105)

		- 可重入锁

		- 公平锁

		- 可中断锁

- [LockSupport](https://segmentfault.com/a/1190000008420938)

	- park

	- upark

- [synchronize](https://upload-images.jianshu.io/upload_images/4491294-e3bcefb2bacea224.png)

	- monitorenter和moniterexit

		- mutex lock实现

	- [优化](https://blog.csdn.net/asiaLIYAZHOU/article/details/76098607)

		- Java对象头MarkWord

			- 无锁 01

			- 偏向锁 01 + ThreadID-> 1 是偏向锁

				- 乐观锁

				- 只有一个线程使用锁

			- 轻量锁 00

				- 乐观锁

			- 重量锁 10

				- 悲观锁

			- GC 11

		- 锁之间的转化

			- [无锁->偏量锁->轻量锁->重量锁](https://www.jianshu.com/p/36eedeb3f912)

			- [转化过程](https://blog.dreamtobe.cn/2015/11/13/java_synchronized/)

				- 一个线程访问的时候

					- 初始化时才CAS，记录ThreadID

				- 第二个线程访问的时候

					- 偏量，且是当前线程(看ThreadID)，直接执行

					- 偏量，且不是当前线程

						- CAS竞争锁，获得锁，置ThreadID

						- CAS竞争失败，升级为轻量级锁

					- 轻量

						- 自旋锁

							- 减少线程阻塞造成的线程切换

							- 顺序拿锁，不允许出现竞争

						- 当前线程的栈帧中创建一个空间用来存储锁记录

							- CAS替换Mark Word

								- 成功进入锁

								- 失败则自旋

							- 执行同步代码，执行完毕后CAS替换Mark Word

								- 成功释放锁

								- 失败升级为重量锁

				- 第三个线程访问的时候

					- 自旋超过一定限制

					- 多个锁竞争

						- 重量级锁

- [CAS](http://www.importnew.com/20472.html)

	- [ABA问题](https://www.cnblogs.com/549294286/p/3766717.html)

		- 版本号

- [AQS](https://www.cnblogs.com/waterystone/p/4920797.html)

	- FIFO队列，可以用于构建锁或者其他相关同步装置的基础框架

	- 状态

		- getState

		- setState

		- compareAndSetState

	- 模式

		- 排他模式

			- acquire

			- release

		- 共享模式

	- [同步器](https://blog.csdn.net/w_s_h_y/article/details/77450166)

		- 线程访问和资源控制

		- 定义了线程对资源是否能够获取以及线程的排队

		- Node
		  双向链表

			- waitStatus

				- CANCELLED

					- 在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该Node的结点

				- SIGNAL

					- 只要前继结点释放锁，就会通知标识为SIGNAL状态的后继结点的线程执行

				- CONDITION

					- 该标识的结点处于等待队列中，结点的线程等待在Condition上

					- 当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中

				- PROPAGATE

					- 与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态

				- 0态

					- 表示初始化状态

			- prev

			- next

			- nextWaiter

				- 存储condition队列中的后继节点

			- thread

			- head

				- head为空结点，不存储信息

			- tail

		- Sync队列

		- Condition队列

### 并行设计模式

- Future 模式

- Master-Worker模式

- 生产者-消费者模式

- ForkJoinPool

	- 把大任务分割成若干个小任务，最终汇总每个小任务结果

	- 工作窃取（work-stealing）

### 协程Fiber

- Quasar

	- 调度器使用ForkJoinPool来调度这些fiber

### 线程池

- newSingleThreadExecutor

- newFixedThreadPool

- newCachedThreadPool

- [线程池的线程数量设置](https://www.zhihu.com/question/38128980)

	- cpu

		- 物理cpu

			- 实际server插槽上cpu的个数

			- cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l

		- 逻辑cpu

			- processor

				- cat /proc/cpuinfo |grep "processor"|wc -l

		- 核数

			- 一般来讲，物理CPU个数×每颗核数=逻辑CPU的个数

			- 如果不相等，说明cpu支持超线程

			- cat /proc/cpuinfo |grep "cores"|uniq

	- 设CPU的processor数为N

		- 如果是CPU密集型应用，则线程池大小设置为N+1

		- 如果是IO密集型应用，则线程池大小设置为2N+1

	- 最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目

- [拒绝策略](http://www.importnew.com/21303.html)

	- AbortPolicy

		- 如果不能接受任务了，则抛出异常

	- CallerRunsPolicy

	- DiscardOldestPolicy

		- 如果不能接受任务，则抛弃一个最老的任务，由一个队里维护

	- DiscardPolicy

		- 如果不能接受任务，则抛弃

### 原理

- Amdahl定律

	- 加速比=优化前系统耗时/优化后系统耗时

	- 增加CPU处理器的数量并不一定能起到有效的作用 提高系统内可并行化的模块比重，合理增加并行处理器数量，才能以最小的投入，得到最大的加速比

- Gustafson定律

	- n + ( 1-n )*p

		- n为串行百分比任务

		- p为加速比

### [线程](http://www.importnew.com/21239.html)

- Thread

	- new

	- runnable

	- terminated

	- blocked

		- synchronized

	- waiting

		- wait/notify

	- time_waiting

- 如何启动一个线程

	- Thread.start

	- Thread.run

		- 实现runnable接口

- 方法

	- interrupt

		- isInterrupted

	- join

		- 等待其他线程结束

	- yeild

		- 把自己的CPU释放掉，然后跟其他线程一起竞争

	- sleep

- 守护线程

- 线程优先级

- FutureTask实现异步

	- FutureTask

	- Futrure

	- 线程池submit

	- [实现Callback<T>](http://www.importnew.com/21312.html)

## Collection

### Map

- TreeMap

	- 红黑树

- [HashMap](https://blog.csdn.net/justloveyou_/article/details/62893086)

	- 链式哈希算法

	- 起始容量(initialCapacity)和装载因子

		- [大于阈值时，rehash，扩容2倍](https://blog.csdn.net/justloveyou_/article/details/62893086)

	- [在多线程下死循环的问题](https://coolshell.cn/articles/9606.html)

		- [rehash可能产生死循环](https://blog.csdn.net/linsongbin1/article/details/54708694)

		- resize时，链表到新位置的顺序与原位置相反，所以可能死循环

	- [JDK1.8解决了死循环问题](http://www.importnew.com/20386.html)

		- resize扩容一倍，且大小是n的2次幂

		- 分为高位地址和地位地址，新增的一个bit为0或1

		- 低位地址rehash的顺序与原位置一致，所以不会产生死循环

- [LinkedHashMap](https://blog.csdn.net/justloveyou_/article/details/71713781)

	- 链式哈希+链表+双向链表

		- before/after指针

	- Node构成一整条双向链表

	- [可实现LRU，重写removeEldestEntry即可](https://crossoverjie.top/%2F2018%2F04%2F07%2Falgorithm%2FLRU-cache%2F?nsukey=j6f%2FOSx73gCHekeiq0uKZfTRra2OmDv8SztIy6zIuiNoKeey7gJhSbF2tungF0TYSxd3TdEG3wMTakoXMtm0itDLyY5qiKo%2BHnq72P35Sh6sIrkJaICAyYKvLQ9kp6u7xn3rK5Cr0Vt9w19PnQSmv2g3JM8eTpiApsjSE%2BWtL2OUNl1NWBb10LxnyjGiyBAVDLkCcjyT7%2F%2FcXGD90MwuJA%3D%3D)

	- access order

		- 保持插入顺序的

		- 保持访问顺序

- [WeekHashMap](https://lijf.me/2017/06/28/from-weakreference-to-weakhashmap/)

	- 弱引用，Entry可能被自动回收

	- Entry的Key为弱引用

	- Key值为若可达时，则可以在GC中被回收，然后弱引用被加入到ReferenceQueue中

	- 遍历ReferenceQueue，非空的话则遍历删除其对应的Entry，达到自动清理过期Entry的效果

	- 清理过期Entry的方法被常用方法get，set等调用，即懒清除

- [IdentityHashMap](https://segmentfault.com/q/1010000002779228)

	- key比较的是“引用相等”

	- 序列化或者深度复制。或者记录对象代理

### List

- ArrayList

	- CopyOnWriteArrayList

		- 写时复制，读写分离、读多写少

- LinkedList

### Queue

- BlockingQueue

- ConcurrentLinkedQueue

## [JMM](https://blog.csdn.net/u011080472/article/details/51337422)

### 定义

- 规定线程和内存之间的关系

- 主内存：Main Memory

	- 变量都储存在主存中，对于所有线程都是共享

	- 对应java堆

	- 存在在主存中

	- load到工作内存

- 工作内存：Working Memory

	- 线程对所有变量的操作都是在工作内存中进行，线程之间无法相互直接访问

	- 虚拟机栈

	- 存储在寄存器、CPU缓存中

	- store到主内存

### 8种操作/原子性

- lock

	- 作用于主内存的变量，把一个变量标识为一条线程独占状态

- unlock

	- 作用于主内存变量，把一个处于锁定状态的变量释放

- read

	- 作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中

- load

	- 作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中

- use

	- 作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎

- assign

	- 作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量

- store

	- 作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中

- write

	- 作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中

### [volatile](https://www.jianshu.com/p/67802b2df675?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

- 内存屏障

- 缓存一致性协议

- [伪共享](https://blog.csdn.net/karamos/article/details/80126704)

	- [缓存行（cache line](https://www.cnblogs.com/RunForLove/p/5624390.html)

		- 当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享

	- 最常见的缓存行大小是64个字节

	- hotspot对象的对象头为8字节

		- 由24位哈希码和8位标志位（如锁的状态或作为锁对象）组成的Mark Word

		- 对象所属类的引用

	- 解决方案

		- JDK1.6缓存行填充

		- JDK1.7把填充放在基类

		- JDK1.8@Contended注解-XX:-RestrictContended

- [happens-before规则](https://www.jianshu.com/p/67802b2df675?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

	- 监视器锁规则

		- 一个unlock操作先行发生于对同一个锁的lock操作

	- volatile变量规则

		- 对一个Volatile变量的写操作先行发生于对这个变量的读操作

	- 程序次序规则

		- 单线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作

	- 线程启动规则

		- Thread对象的start()方法先行发生于此线程的其他动作

	- 线程中断规则

		- 对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生

	- 线程终止规则

		- 线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行

	- 对象终结规则

		- 一个对象的初始化完成先行发生于它的finalize()方法的开始

	- 传递规则

### [final](http://www.infoq.com/cn/articles/java-memory-model-6)

- 禁止重排序

	- 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序

		- JMM禁止编译器把final域的写重排序到构造函数之外

		- 编译器会在final域的写之后，构造函数return之前，插入一个StoreStore屏障

	- 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序

		- 在一个线程中，初次读对象引用与初次读该对象包含的final域，JMM禁止处理器重排序这两个操作

		- 编译器会在读final域操作的前面插入一个LoadLoad屏障

- 原因：final引用不能从构造函数内“逸出”

### 重排序

- 分类

	- 编译器优化的重排序

	- 指令级并行的重排序

		- CPU采用了允许将多条指令不按照程序规定的顺序，分开发送给各个相应电路单元处理

	- 内存系统的重排序

- 内存屏障

	- 编译器优化的重排序

		- 重排序规则

	- 指令级并行的重排序

		- Java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序

		- 指令分类

			- LoadLoad Barriers

			- StoreStore Barriers

			- LoadStore Barriers

			- StoreLoad Barriers

		- 具有数据依赖性的禁止重排序

			- 写后读

			- 写后写

			- 读后写

		- as-if-serial

			- 不管怎么重排序，单线程的处理结果不能被改变

	- 内存系统的重排序

### long/double非原子协定

- 2个能存储32位变量的变量槽

- 本质上不具有原子性，但JVM可以实现为原子的

### 特性

- 原子性

	- synchronize

	- lock/unlock

- 可见性

	- volatile

	- synchronize

	- final

- 有序性

	- as-if-serial

		- 如果在本线程内观察，所有的操作都是有序的

	- 如果在一个线程中观察另一个线程，所有的操作都是无序的

## 反射技术

### 动态代理技术

- [原理](https://juejin.im/post/5a99048a6fb9a028d5668e62)

- 分类

	- [动态代理](https://blog.csdn.net/luanlouis/article/details/24589193)

		- JDK

			- newProxyInstance

		- ASM

		- CGLIB

			- 无论目标对象有没有实现接口都可以代理，但是无法处理final的情况

			- 在代码里生成字节码，并动态地加载成class对象、创建实例

		- Javassist

		- Javassist动态字节码生成

		- [对比](https://blog.csdn.net/luanlouis/article/details/24589193)

		- [对比2](https://www.cnkirito.moe/rpc-dynamic-proxy/)

		- 区别

			- java动态代理

				- 通过接口中的方法名，在动态生成的代理类中调用业务实现类的同名方法

			- CGLib代理

				- 通过继承业务类，生成的动态代理类是业务类的子类，通过重写业务方法进行代理

	- [静态代理](http://javatar.iteye.com/blog/814426)

		- 代理模式

			- use

			- interface

- 应用

	- AOP

		- 事务提交或回退

		- 权限管理

		- 自定义缓存逻辑处理

		- 日志处理

## IO

### [Java NIO](https://www.cnblogs.com/geason/p/5774096.html)

## JVM

### 垃圾回收

- finalize

- 内存区域

	- 程序计数器

	- java虚拟机栈

		- StackOverflowError

		- OutOfMemoryError

	- 本地方法栈

		- StackOverflowError

		- OutOfMemoryError

	- java堆(GC堆)

		- 新生代

			- Eden

			- FromSurvivor

			- To Survivor

			- 详情

				- [Eden(8) Survivor(1)](http://www.cnblogs.com/duanxz/p/6076662.html)

				- 为什么要有survivor

					- 减少被送到老年代的对象，进而减少Full GC的发生，Survivor的预筛选保证，只有经历16次Minor GC还能在新生代中存活的对象，才会被送到老年代

				- 为什么要有两个Survivor

					- 解决碎片化问题，使用到复制算法

		- 老年代（Tenured）

	- 方法区

		- 永久代（Perm）

- [垃圾收集器](https://blog.csdn.net/tjiyu/article/details/53983650)

	- 新生代收集器(复制算法)

		- Serial

			- 单线程

				- Stop The World

				- 单CPU场景或桌面应用

		- ParNew

			- 并行

		- Parallel Scavenge

			- 吞吐量优先

	- 老年代收集器

		- SerialOld

			- 标记-整理

		- Parallel Old

			- 标记-整理

		- [CMS](http://www.cnblogs.com/rgever/p/9534857.html)

			- 标记-清除

				- 浮动垃圾

				- "Concurrent Mode Failure"失败

				- 大量内存碎片

					- 可能导致提前触发full gc

			- 步骤

				- 初始标记

				- 并发标记

				- 最终标记

				- 并发清除

			- 并发收集、低停顿

	- 整堆收集器

		- [G1](https://www.jianshu.com/p/bdd6f03923d1)

			- 局部：复制

			- 整体：标记-整理

			- 特点

				- 能充分利用CPU、多核环境下的硬件优势，使用多个CPU（CPU或者CPU核心）来缩短stop-The-World停顿时间

				- 可预测的停顿

				- G1将内存分成多个等大小的region，Eden/ Survivor/Old分别是一部分region的逻辑集合，物理上内存地址并不连续

			- 步骤

				- 初始标记

				- 并发标记

				- 最终标记

				- 筛选回收

- GC类型

	- 年轻代GC Minor GC

	- 老年代GC Full GC

- GC日志

	- -XX:+PrintGC

- [引用类型](http://blog.51cto.com/zhangjunhd/53092)

	- 强引用

	- 软引用

		- 符合垃圾收集，只有在内存不足时回收

		- 对象缓存

	- 弱引用

		- 符合垃圾收集，JVM垃圾回收都回收

		- 对象缓存

		- 回调函数中防止内存泄露

		- 回收步骤

			- 弱引用对象不会阻止其指向的对象被GC回收

			- 当一个对象是弱可达的，那么GC会原子地清除所有指向该对象的弱引用，并标记该对象为finalizable并在随后将其回收

			- 刚被清除的弱引用会被加入到ReferenceQueue中

			- 所以可以自动清理过期的Entry值

	- 虚引用

		- 能在对象被GC时收到系统通知

- 回收时间

	- 强制调用System.gc

	- minor gc

		- 新生代的Eden区满了

	- full gc

		- 老生代空间不足，如使用大数组

		- 永久代空间满，未采用CMS GC会引发Full gc

		- 通过MinorGC到老年代的对象大于老年代的剩余空间

		- 由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

	- gc与非gc时间耗时超过了GCTimeRatio的限制引发OOM

- 回收对象

	- 可达性分析法搜索不到的对象

	- 是否需要执行finalize(finalize只能被执行一次)

		- 否，则直接回收

		- 是，则进入FQueue队列，执行完finalize后释放（但如果执行finalize过程中被引用了，则移出FQueue，移出后不再执行finalize）

### [类加载器](https://www.cnblogs.com/novalist/p/6603669.html)

- 分类

	- Bootstrap类加载器（lib）

		- 本地C++实现

		- System.class.getClassLoader()结果为null

	- Extension类加载器（lib/ext）

	- Application类加载器(系统类路径(CLASSPATH)

	- 上下文(Custom)类加载器

- 双亲委派机制描述 

	- 某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成类加载任务，就成功返回；只有父类加载器无法完成此加载任务时，才自己去加载

	- 目的：防止内存中出现多份同样的字节码

### [类初始化](https://www.cnblogs.com/zhguang/p/3154584.html)

- 字节码

	- 只要字节码符合java指令集和文件格式，就可以在JVM下运行

		- java

		- Scala

- [装载过程](https://blog.csdn.net/wen7280/article/details/53856790/)

	- 加载（JVM必须先完成）

		- 通过类的全名，获取类的二进制数据流

		- 解析类的二进制数据流为方法区内的数据结构，也就是将类文件放入方法区中

		- 创建java.lang.Class类的实例，表示该类型

	- 链接

		- 验证

			- 主要目的是保证加载的字节码是符合规范的

			- 格式检查

				- 魔数检查

				- 版本检查

				- 长度检查

			- 语义检查

				- 抽象方法是否实现

				- 是否继承final

				- 是否有父类

			- 字节码检查

				- 跳转命令是否执向正确位置

				- 操作类型是否合理

			- 符号引用检查

		- 准备

			- 类变量（static修饰的变量）分配内存并设置类变量初始值

			- 类变量static，不包括实例变量

			- 为类变量设置初始值是设为其数据类型的“零值”

			- 在准备阶段，不会有任何java代码被执行

		- 解析

			- 将类、接口、字段和方法的符号引用转为直接引用

			- 得到类或者字段、方法在内存中的指针或者偏移量

	- 初始化

		- 开始执行java字节码

		- clinit()

			- 由类静态成员的赋值语句以及static语句块合并产生的

		- 虚拟机会保证在子类的clinit()方法执行之前，父类的clinit()方法已经执行完毕

		- 初始化类的过程必须保持同步，如果有多个线程初始化一个类，仅仅允许一个线程执行初始化，其他的线程都需要等待

- 初始化条件

	- 一个类或者接口在初次使用时

		- 创建一个类的实例

		- 当调用类的静态方法时

		- 当使用类或者接口的静态字段时，除去final字段

		- 当使用java.lang.reflect包中的方法反射类的方法

		- 当初始化子类时，必须先初始化父类

		- 作为启动虚拟机、含有main方法的那个类

	- 其他情况均属于被动使用

		- 被动使用并不会初始化，加载了类，并没有初始化

		- 在引用一个字段时，只有直接定义该字段的类，才会被初始化

	- 这种特征叫动态加载

- 实例初始化

	- 父类构造函数init调用先于子类构造器init函数调用

	- 在执行构造器init函数首先会初始化类中成员变量或者执行非静态代码块（这二者执行的先后顺序依赖于在源文件中出现的顺序）

	- 最后再调用构造函数

- [线程安全](https://blog.csdn.net/duanxin8678/article/details/39313791)

	- 不可变的

		- 如String，在创建实例时提供信息，整个生命期间内固定不变

			- 声明为final

	- 无条件的线程安全

		- 如ConcurrentHashMap和Random

	- 有条件的线程安全

		- Hashtable、Vector等

	- 非线程安全

		- HashMap等

	- 私有锁 private finnal

		- 可以避免子类和基类中同步方法相互干扰的问题

	- [延迟初始化](https://www.jianshu.com/p/47649b1fdbf8)

		- 静态域

		- synchronize关键字修饰

		- 延迟初始化持有类模式，即静态内部类静态初始化

			- private static class FieldHolder

			- 静态内部类在使用的时候才进行初始化

		- 实例域双重检查模式

			- synchronize+两次null检查

			- [声明为volatile](https://www.jianshu.com/p/4b6cf8f10950)

		- [实例域单重检查模式](https://blog.csdn.net/a921122/article/details/54748691)

			- 必须为可接受重复初始化

			- 声明为volatile

		- [含有final域的对象](http://www.voidcn.com/article/p-mmxogupa-boa.html)

## 基础

### [==和equals和HashCode的区别](https://blog.csdn.net/justloveyou_/article/details/52464440)

### 泛型

- [ParameterizedType](https://www.jianshu.com/p/b1ad2f1d3e3e)

### 数据类型

- String

	- 不能被继承，设计成final

		- 主要是为了 “ 效率 ” 和 “ 安全性 ” 的缘故。 若 String 允许被继承, 由于它的高度被使用率, 可能会降低程序的性能，所以 String 被定义成 final

### 关键字

- static

	- static块初始化是线程安全的

### 语法糖

- 内部类

- 自动拆箱/装箱

- 泛型技术

- 枚举

- foreach循环

- 变长参数

- 条件编译

- 对枚举和字符串的switch支持

- [JDK1.7：自动释放资源接口](https://yq.aliyun.com/articles/43519)

	- try-with-resource

## 调优

### 启动参数

- -Xmx

	- JVM最大可用内存

- -Xms

	- JVM初始化内存

- -Xmn

	- 年轻代最大堆内存

- -Xss

	- 每个线程的堆栈大小

- -XX:NewRatio

	- 年轻代与年老代的比值

- -XX:SurvivorRatio

	- 年轻代中Eden和Survivor的比值

- -XX:MaxPermSize

	- 持久代大小

- -XX:MaxTenuringThreshold

	- 年轻代到年老带的最大迭代数

- XX:+UseConcMarkSweepGC

	- 设置年老代为CMS收集算法

	- -XX:CMSFullGCsBeforeCompaction

		- 多少次GC之后对内存进行压缩整理，避免内存碎片

- -XX:+UseParNewGC

	- 设置年轻代为并行收集算法

- -XX:+PrintGC

	- 打印GC日志

- -XX:+PrintGCDetails

