# Java编程基础


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

			- From&nbsp;Survivor

			- To&nbsp; Survivor

			- 详情

				- [Eden(8) Survivor(1)](http://www.cnblogs.com/duanxz/p/6076662.html)

				- 为什么要有survivor

					- 减少被送到老年代的对象，进而减少Full GC的发生，Survivor的预筛选保证，只有经历16次Minor GC还能在新生代中存活的对象，才会被送到老年代

				- 为什么要有两个Survivor

					- 解决碎片化问题，使用到复制算法

		- 老年代（Tenured）

	- 方法区

		- Jdk1.8之前 永久代（Perm）

		- Jdk1.8之后 元空间(metaSpace)

- 对象的内存布局

	- 对象头(Header)

		- Mark Word：对象运行时的数据，如哈希码、GC分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳

		- 类型指针

			- 执行类的元数据指针

			- 如果是数值，还有一块存放数组的长度

	- 实例数据

		- 子类、父类定义的字段内容

	- 填充数据

- 对象的访问

	![句柄指针](https://user-gold-cdn.xitu.io/2017/9/4/de6924b6e9d576105ba24700f1f357f4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)  
	

	- Java 堆中会分配一块内存作为句柄池

	- 先通过句柄，句柄再指向对象的直接地址指针

- [垃圾收集器](https://blog.csdn.net/tjiyu/article/details/53983650)

	- 并发和并行的区别

		- 并发 Concurrency

			- 一个并发程序是具备处理多个任务的能力。并发并不需要有多个CPU,单个CPU通过时间片的方式,不同时间片处理不同任务,可以让程序“看起来”是都在执行的

		- 并行 Parallelism

			- 并行表示在同一个时间点,有多个任务都在进行当中。并行是需要多个CPU才可以完成的。如果说每一个CPU都在运行一个程序,那么4个CPU就可以让4个程序“并行”运算

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

		- Serial&nbsp;Old

			- 标记-整理

		- Parallel Old

			- 标记-整理

		- [CMS](http://www.cnblogs.com/rgever/p/9534857.html)

			- 标记-清除

				- 浮动垃圾

				- &quot;Concurrent Mode Failure&quot;失败

					- 清理工作还没有开始，old gen已经没有空间容纳更多对象了

					- CMSInitiatingOccupancyFraction提前触发CMS

				- 大量内存碎片

					- 可能导致提前触发full gc

					- 减少内存碎片

						- CMSFullGCsBeforeCompaction执行多少次full GC之后做压缩

				- CMS G1

			- [步骤](https://blog.csdn.net/zqz_zqz/article/details/70568819)

				![CMS步骤](https://user-gold-cdn.xitu.io/2017/9/4/6f4d683644a154537b3e23d60d49c074?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)  
				

				- 标记

					- 对象在标记过程中，根据标记情况，分成三类：&lt;br&gt;&lt;br&gt;白色对象，表示自身未被标记；&lt;br&gt;灰色对象，表示自身被标记，但内部引用未被处理；&lt;br&gt;黑色对象，表示自身被标记，内部引用都被处理；

				- [初始标记](http://www.importnew.com/27822.html)

					- 标记GC Roots可达的老年代对象

					- 遍历新生代对象，标记可达的老年代对象

				- 并发标记

					- 该阶段GC线程和应用线程并发执行

					- 遍历InitialMarking阶段标记出来的存活对象，然后继续递归标记这些对象可达的对象（遍历内部引用）

					- 为了提高重新标记的效率，该阶段会把上述对象所在的Card标识为Dirty，后续只需要扫描Dirty的对象，而不需要整个老年代

				- 预清理

					- 处理前一个阶段因为引用关系改变导致没有标记到的存活对象的

				- 可被终止的预清理

					- 处理 From 和 To 区的对象，标记可达的老年代对象&lt;br&gt;和上一个阶段一样，扫描处理Dirty Card中的对象

				- 最终标记

					- 修正并发标记期间的变动部分

				- 并发清除

				- 并发重置

			- 并发收集、低停顿，吞吐量优先的垃圾收集器

			- -”XX:+UseConcMarkSweepGC”

	- 整堆收集器

		- [G1](https://www.jianshu.com/p/bdd6f03923d1)

			![堆内存](http://upload-images.jianshu.io/upload_images/2184951-715388c6f6799bd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)  
			

			- 局部：复制

			- 整体：标记-整理

			- 内存块（region）

				- E

				- S

				- O

				- H

					- Region存储的是巨型对象（humongous object，H-obj），当新建对象大小超过Region大小一半时，直接在新的一个或多个连续Region中分配，并标记为H

				- 大小为1M、2M、4M、8M、16M和32M等，默认2048个

				- G1将内存分成多个等大小的region，Eden/ Survivor/Old分别是一部分region的逻辑集合，物理上内存地址并不连续

			- 特点

				- 能充分利用CPU、多核环境下的硬件优势，使用多个CPU（CPU或者CPU核心）来缩短stop-The-World停顿时间

				- 可预测的停顿

			- gc

				- young gc&lt;br&gt;

					- 所有eden region被耗尽无法申请内存时，就会触发一次young gc

				- mixed gc

					- 步骤

						- 初始标记

						- 并发标记

						- 最终标记

						- 筛选回收

				- full gc

					- 如果对象内存分配速度过快，mixed gc来不及回收，导致老年代被填满，就会触发一次full gc

					- 单线程执行的serial old gc，会导致异常长时间的暂停时间

					- 需要尽量避免old gc

- GC类型

	- 年轻代GC Minor GC

	- 老年代GC Full GC

- GC日志

	- -XX:+PrintGC&nbsp; -XX:+PrintGCDetails

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

	- 什么时候发生full gc？

		- 老生代空间不足，如使用大数组

		- 永久代空间满，未采用CMS GC会引发Full gc

		- 通过MinorGC到老年代的对象大于老年代的剩余空间

		- 由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

	- gc与非gc时间耗时超过了GCTimeRatio的限制引发OOM

- Stop the world

- [safe point](https://www.ezlippi.com/blog/2018/01/safepoint.html)

	- 代码中一些特定的位置,当线程运行到这些位置时它的状态是确定的,这样JVM就可以安全的进行一些操作

		方法返回之前 调用某个方法之后 抛出异常的位置 循环的末尾  
		

	- JVM在做GC之前要等所有的应用线程进入到安全点后VM线程才能分派GC任务

	- 使用场景

		- 垃圾回收(这是最常见的场景)

		- 取消偏向锁(JVM会使用偏向锁来优化锁的获取过程)

		- Class重定义(比如常见的hotswap和instrumentation)

		- Code Cache Flushing(JDK1.8在CodeCache满的情况下就可能出现)

		- 线程堆栈转储(jstack命令)

	- -XX:+PrintGCApplicationStoppedTime这个参数来打印JVM暂停的时间

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

### 方法调用指令

- invokestatic ：调用静态方法

- invokespecial ：调用私有方法、构造器方法、父类方法

- invokevirtual ：调用虚方法

- invokeinterface ：调用接口方法

- invokedynamic ：先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法

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

		- &lt;clinit&gt;()

			- 由类静态成员的赋值语句以及static语句块合并产生的

		- 虚拟机会保证在子类的&lt;clinit&gt;()方法执行之前，父类的&lt;clinit&gt;()方法已经执行完毕

		- 线程安全：初始化类的过程必须保持同步，如果有多个线程初始化一个类，仅仅允许一个线程执行初始化，其他的线程都需要等待

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

	- 父类构造函数&lt;init&gt;调用先于子类构造器&lt;init&gt;函数调用

	- 在执行构造器&lt;init&gt;函数首先会初始化类中成员变量或者执行非静态代码块（这二者执行的先后顺序依赖于在源文件中出现的顺序）

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

- [final](https://www.jianshu.com/p/f68d6ef2dcf0)

	- 修饰类（Class）

		- 无法被继承

		- 类中的方法默认都是final 修饰的

	- 修饰方法（Method）

		- 不能被子类重写或者覆盖

		- 在编译期间会自动做内联优化

	- 修饰域（Field）

		- 出现在常量池中，类似宏

		- 并发：内存可见性

	- 修饰方法参数（Method Argument）

		- 匿名类中，形参用final&nbsp;

		- 实现上是拷贝引用，为了避免引用值发生改变，例如被外部类的方法修改等，而导致内部类得到的值不一致，于是用final来让该引用不可改

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

- [ThreadLocalMap的Key值为弱引用，即WeakReference&lt;ThreadLocal&lt;?&gt;&gt;](https://majiaji.coding.me/2017/03/27/threadLocal-WeakReference%E5%92%8C%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E7%9A%84%E6%80%9D%E8%80%83/)

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

			- 偏向锁 01 + ThreadID-&gt; 1 是偏向锁

				- 乐观锁

				- 只有一个线程使用锁

			- 轻量锁 00

				- 乐观锁

			- 重量锁 10

				- 悲观锁

			- GC 11

		- 锁之间的转化

			- [无锁-&gt;偏量锁-&gt;轻量锁-&gt;重量锁](https://www.jianshu.com/p/36eedeb3f912)

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

	- [模式](http://ifeve.com/introduce-abstractqueuedsynchronizer/)

		- 排他模式

			- acquire

				- while(获取锁) {&lt;br&gt; if (获取到) {&lt;br&gt;  退出while循环&lt;br&gt; } else {&lt;br&gt;  if(当前线程没有入队列) {&lt;br&gt;   那么入队列&lt;br&gt;  }&lt;br&gt;  阻塞当前线程&lt;br&gt; }&lt;br&gt;}

			- release

				- if (释放成功) {&lt;br&gt; 删除头结点&lt;br&gt; 激活原头结点的后继节点&lt;br&gt;}

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

	- 3.2CompletableFuture

- Master-Worker模式

- 生产者-消费者模式

- ForkJoinPool

	![ForkJoin线程池](https://user-gold-cdn.xitu.io/2018/9/11/165c91a991b0fbad?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)  
	

	- 把大任务分割成若干个小任务，最终汇总每个小任务结果

	- 工作窃取（work-stealing）

	- 子类

		- RecursiveAction&nbsp;含任务执行结果

		- RecursiveTask 不含任务执行结果

- CountDownLatch

	- 先划分子任务，CountDownLatch等待子任务运行完毕

- parallelStream

- 分片

### 协程Fiber

- Quasar

	- 调度器使用ForkJoinPool来调度这些fiber

### 线程池

- [分类](https://blog.csdn.net/u011974987/article/details/51027795)

	- newCachedThreadPool

		- 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程

		- new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue&lt;Runnable&gt;())

	- newFixedThreadPool

		- 线程数量固定，超出放在阻塞队列中(LinkedBlockingQueue)。创建的线程不回收

		- new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue&lt;Runnable&gt;())

	- newSingleThreadExecutor

		- 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行

		- new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue&lt;Runnable&gt;())

	- newScheduledThreadPool

		- super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue())

		- 创建一个定长线程池，支持定时及周期性任务执行

		- 比Timer更安全，功能更强

- [ThreadPoolExecutor](https://juejin.im/entry/58fada5d570c350058d3aaad)

	![线程池处理策略](https://user-gold-cdn.xitu.io/2017/4/22/b17448d868e81c5a53c419a70d3fe59e?imageslim)  
	

	- 参数

		- corePoolSize

			- 核心线程数量

				当有新任务在execute()方法提交时，会执行以下判断： 1. 如果运行的线程少于 corePoolSize，则创建新线程来处理任务，即使线程池中的其他线程是空闲的； 2. 如果线程池中的线程数量大于等于 corePoolSize 且小于 maximumPoolSize，则只有当workQueue满时才创建新的线程去处理任务； 3. 如果设置的corePoolSize 和 maximumPoolSize相同，则创建的线程池的大小是固定的，这时如果有新任务提交，若workQueue未满，则将请求放入workQueue中，等待有空闲的线程去从workQueue中取任务并处理； 4. 如果运行的线程数量大于等于maximumPoolSize，这时如果workQueue已经满了，则通过handler所指定的策略来处理任务  
				

			- 任务提交时，判断的顺序为 corePoolSize --&gt; workQueue --&gt; maximumPoolSize

		- maximumPoolSize

			- 最大线程数

		- workQueue

			- 等待队列，当任务提交时，如果线程池中的线程数量大于等于corePoolSize的时候，把该任务封装成一个Worker对象放入等待队列

			- 分类

				- 直接切换

					- SynchronousQueue

				- 使用无界队列

					- LinkedBlockingQueue

				- 使用有界队列

					- ArrayBlockingQueue

					- 如果要想降低系统资源的消耗，可以采用低CorePoolSize，高队列长度

					- 如果提交的任务经常发生阻塞，那么可以考虑通过调用 setMaximumPoolSize() 方法来重新设定线程池的容量

					- 如果队列的容量设置的较小，通常需要将线程池的容量设置大一点，以提交CPU的利用率

		- keepAliveTime

			- 线程池维护线程所允许的空闲时间

			- 当线程池中的线程数量大于corePoolSize的时候，如果没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待到超过keepAliveTime

		- threadFactory

			- 创建新线程

			- Executors.defaultThreadFactory() 

		- handler

			- 线程池的饱和策略

	- 数据结构

		- HashSet&lt;Worker&gt; workers 

			![worker运行状态](https://user-gold-cdn.xitu.io/2017/4/22/fb66e616aa6da061f92d0b4eebb64b7f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)  
			

			- 实现了Runnable和AQS

			- firstTask

				- 保存传入的任务并运行

			- 线程池新建的线程运行的是Worker

			- 其他时间去等待队列获取任务并执行

			- 为什么使用AQS

				- 灵活控制，比如ReentrantLock是可重入的，这里需要不可重入

					之所以设置为不可重入，是因为我们不希望任务在调用像setCorePoolSize这样的线程池控制方法时重新获取锁。如果使用ReentrantLock，它是可重入的，这样如果在任务中调用了如setCorePoolSize这类线程池控制的方法，会中断正在运行的线程  
					

				- 一旦获取了独占锁，表示当前线程正在执行任务，不可中断

				- 如果该线程现在不是独占锁的状态，也就是空闲的状态，说明它没有在处理任务，这时可以对该线程进行中断

- 状态

	![线程池状态](https://user-gold-cdn.xitu.io/2017/4/22/77c2251dacf9e26fe1ba1e1e54e62a65?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)  
	

	- RUNNING

	- SHUTDOWN

		- 不再接受新提交的任务，但却可以继续处理阻塞队列中已保存的任务

		- shutdown

	- STOP

		- 不能接受新任务，也不处理队列中的任务，会中断正在处理任务的线程

		- shutdownNow

	- TIDYING

		- 如果所有的任务都已终止了，workerCount (有效线程数) 为0，线程池进入该状态后会调用 terminated() 方法进入TERMINATED 状态

	- TERMINATED

		- 在terminated() 方法执行完后进入该状态，默认terminated()方法中什么也没有做

- [Excutors](https://yemengying.com/2017/03/17/difference-between-executor-executorService/)

	- 提供工厂方法来创建不同类型的线程池

	- 不适合自定线程某些参数的场景，如设置拒绝策略

- [线程池的线程数量设置](https://www.zhihu.com/question/38128980)

	- cpu

		- 物理cpu

			- 实际server插槽上cpu的个数

			- cat /proc/cpuinfo |grep &quot;physical id&quot;|sort |uniq|wc -l

		- 逻辑cpu

			- processor

				- cat /proc/cpuinfo |grep &quot;processor&quot;|wc -l

		- 核数

			- 一般来讲，物理CPU个数×每颗核数=逻辑CPU的个数

			- 如果不相等，说明cpu支持超线程

			- cat /proc/cpuinfo |grep &quot;cores&quot;|uniq

	- 设CPU的processor数为N

		- 如果是CPU密集型应用，则线程池大小设置为N+1

		- 如果是IO密集型应用，则线程池大小设置为2N+1

	- 最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目

- [拒绝策略](http://www.importnew.com/21303.html)

	- AbortPolicy

		- 如果不能接受任务了，则抛出异常

	- CallerRunsPolicy

		- 用调用者的线程来运行任务

	- DiscardOldestPolicy

		- 如果不能接受任务，则抛弃一个最老的任务，由一个队列维护

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

		- 等待其他线程结束，并不释放对象锁

	- yeild

		- 把自己的CPU释放掉，然后跟其他线程一起竞争，并不释放对象锁

	- sleep

		- 使当前线程（即调用该方法的线程）暂停执行一段时间，让其他线程有机会继续执行，但它并不释放对象锁

- [对象](http://dylanxu.iteye.com/blog/1322066)

	- synchronize

	- wait

		- 使当前线程暂停执行并释放对象锁标示，让其他线程可以进入synchronized数据块

	- notify

		- 将从对象的等待池中移走一个任意的线程并放到锁标志等待池中

	- notifyall

		- 从对象等待池中移走所有等待那个对象的线程并放到锁标志等待池中

	- 必须在synchronized语句块内使用

- [线程转态转移图](https://www.cnblogs.com/happy-coder/p/6587092.html)

	![线程状态转移图](https://images2015.cnblogs.com/blog/716271/201703/716271-20170320112245721-1831918220.jpg)  
	

- [公平锁和非公平锁](https://blog.csdn.net/qyp199312/article/details/70598480)

	- 公平锁

		- 每次有线程来抢占锁的时候，都会检查一遍有没有等待队列，如果有放在等待队列

	- 非公平锁

		- 非公平锁在实现的时候则不用检查，强调随机抢占

- 守护线程

- 线程优先级

- FutureTask实现异步

	- FutureTask

	- Futrure

	- 线程池submit

	- [实现Callback&lt;T&gt;](http://www.importnew.com/21312.html)

- native thread vs green thread

	- 在 Linux 中，每一个 Java thread 对应一个 native thread

	- hotspot等主流jvm实现的是本地线程

	- 使用线程池可以通过重用有限数量的线程来降低重新生成新线程的成本

	- java绿色线程只在一些开源jvm项目中支持

- 线程调度器

	- 线程的调度自然是由 Linux kernel 进行

	- Linux 使用轻量化进程（lightweight process）对多线程应用提供更好的支持

	- 调度方案

		- 抢占式调度

			- Java中线程会按优先级分配CPU时间片运行

			- 优先级 1~10

		- 协同式调度

			- 线程的执行时间由线程本身控制，线程把自己的工作执行完之后，要主动通知系统切换到另外一个线程上&lt;br&gt;

			- java线程优先级和操作系统之间不一定是一一对应的

			- 一条Java线程就映射到一条轻量级进程

	- [实现线程的方式](https://blog.csdn.net/nalanmingdian/article/details/77748326)

		- 内核线程

			- 直接由操作系统内核（Kernel，即内核）支持的线程，一对一

			- 轻量级进程（LWP）

		- 用户线程

			- 完全建立在用户空间的线程库上

			- 1：N

		- 用户线程加轻量级进程混合实现

			- N:M

### [延迟任务/定时任务](https://blog.csdn.net/xybelieve1990/article/details/78040419)

- Timer

	- schedule(TimerTask task, long delay, long period)

	- TimerTask类用于实现由Timer安排的一次或重复执行的某个任务。每一个Timer对象对应的是一个线程

	- 无法指定时间运行

- ScheduledExecutorService

- DelayQueue

	- 放置实现了Delay接口的对象，其中的对象只能在其到期时才能从队列中取走

	- 存在分布式横向扩展的问题，因为数据都是存在内存中

- [时间轮(HashedWheelTimer)](https://segmentfault.com/a/1190000010987765)

	![时间轮算法](http://7xlnw9.com1.z0.glb.clouddn.com/netty-hasedwheeltimer1.png)  
	

	- 环形结构(槽位，rounds)，链式hash

	- 根据超时时间的 hash 值(这个 hash 值实际上就是ticks &amp; mask)将 task 分布到不同的槽位中

	- 添加/删除 O(1), 执行 最差O(n)

	- ticksPerWheel（一轮的tick数），tickDuration（一个tick的持续时间）以及 timeUnit（时间单位）

	- [任务执行是串行的，前面任务可能影响到后面的任务的运行](https://zacard.net/2016/12/02/netty-hashedwheeltimer/)

- Redis ZSet

- Spring Task

- Quartz

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

- null值支持情况

	|集合类 |Key |Value| Super |说明 | | -------- | -----: | :----: |:----: |:----: | |Hashtable |不允许为 null |不允许为 null |Dictionary |线程安全 | |ConcurrentHashMap| 不允许为 null |不允许为 null| AbstractMap |分段锁技术| |TreeMap |不允许为 null |允许为 null |AbstractMap |线程不安全 | |HashMap |允许为 null| 允许为 null| AbstractMap| 线程不安全|  
	

### List

- ArrayList

	- 当ArrayList扩容的时候，首先会设置新的存储能力为原来的1.5倍

- LinkedList

- CopyOnWriteArrayList

	- 写时复制，读写分离、读多写少

### Queue

- BlockingQueue

	- ArrayBlockingQueue

	- LinkedBlockingQueue

	- PriorityBlockingQueue

	- LinkedTransferQueue

		- ConcurrentLinkedQueue、SynchronousQueue (公平模式下)、无界的LinkedBlockingQueues等的超集。

		- transfer

			- 若当前存在一个正在等待获取的消费者线程，即立刻移交之；否则，会插入当前元素e到队列尾部，并且等待进入阻塞状态，到有消费者线程取走该元素

	- SynchronousQueue

		- 特点

			- 类似于&quot;单工模式&quot;,对于每个put/offer操作,必须等待一个take/poll操作,类似于我们的现实生活中的&quot;火把传递&quot;:

			- 对于无法进入队列的元素,需要有额外的&quot;拒绝&quot;策略支持.

		- 应用

			- newCachedThreadPool

	- DelayQueue

- Queue

	- LinkedList

		- 双向链表

	- ArrayDeque

		- 循环数组

	- PriorityQueue

		- 堆

- ConcurrentLinkedQueue

### 结构图

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

	- 在每个volatile写操作的前面插入一个StoreStore屏障

	- 在每个volatile写操作的后面插入一个SotreLoad屏障

	- 在每个volatile读操作的后面插入一个LoadLoad屏障

	- 在每个volatile读操作的后面插入一个LoadStore屏障

	- 写

		- 所有Volatile变量写操作之前的针对其他任何变量的读写操作，都不会被编译器、CPU优化后，乱序到Volatile变量的写操作之后执行。

	- 读

		- 带有Acquire语义，所有Volatile变量读操作之后的针对其他任何变量的读写操作，都不会被编译器、CPU优化后，乱序到volatile变量的读操作之前进行。

- 缓存一致性协议

- [伪共享](https://blog.csdn.net/karamos/article/details/80126704)

	- [缓存行（cache line](https://www.cnblogs.com/RunForLove/p/5624390.html)

		- 当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享

	- 最常见的缓存行大小是64个字节

	- hotspot对象的对象头为8字节

		- 由24位哈希码和8位标志位（如锁的状态或作为锁对象）组成的Mark Word

		- 对象所属类的引用

	- 解决方案

		- JDK1.6&nbsp;缓存行填充

		- JDK1.7&nbsp;把填充放在基类

		- JDK1.8&nbsp;@Contended注解&nbsp;-XX:-RestrictContended

- [happens-before规则](https://www.jianshu.com/p/67802b2df675?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

	- 监视器锁规则

		- 一个unlock操作先行发生于对同一个锁的lock操作

	- volatile变量规则

		- 对一个Volatile变量的写操作先行发生于对这个变量的读操作

	- 线程启动规则

		- Thread对象的start()方法先行发生于此线程的其他动作

	- 线程中断规则

		- 对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生

	- 线程终止规则

		- 线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行

	- 对象终结规则

		- 一个对象的初始化完成先行发生于它的finalize()方法的开始

	- 传递规则

	- 程序次序规则

		- 单线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作

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

### long/double非原子协定&lt;br&gt;

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

