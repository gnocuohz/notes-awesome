## 一、JAVA基础
##### 1. JAVA中的几种基本数据类型是什么，各自占用多少字节。

##### 2. String类能被继承吗，为什么。

##### 3. String，Stringbuffer，StringBuilder的区别。

##### 4. ArrayList和LinkedList有什么区别。

##### 5. 讲讲类的实例化顺序，比如父类静态数据，构造函数，字段，子类静态数据，构造函数，字 段，当new的时候，他们的执行顺序。

##### 6. 用过哪些Map类，都有什么区别，HashMap是线程安全的吗,并发下使用的Map是什么，他们内部原理分别是什么，比如存储方式，hashcode，扩容，默认容量等。

##### 7. JAVA8的ConcurrentHashMap为什么放弃了分段锁，有什么问题吗，如果你来设计，你如何设计。

##### 8. 有没有有顺序的Map实现类，如果有，他们是怎么保证有序的。

##### 9. 抽象类和接口的区别，类可以继承多个类么，接口可以继承多个接口么,类可以实现多个接口么。

##### 10. 继承和聚合的区别在哪。

##### 11. IO模型有哪些，讲讲你理解的nio ，他和bio，aio的区别是啥，谈谈reactor模型。

##### 12. 反射的原理，反射创建类实例的三种方式是什么。

##### 13. 反射中，Class.forName和ClassLoader区别 。
> Class.forName会初始化，lassLoader只加载。

##### 14. 描述动态代理的几种实现方式，分别说出相应的优缺点。  
> JDK动态代理、CGLIB(基于ASM包装)。  
> ASM和JAVAASSIST都可以生成字节码实现动态代理，JAVAASSIST是Java源码（本身提供的动态代理接口最慢，比JDK自带的还慢），ASM使用需要手工写字节码。  
> 而aspectj只是字节码织入，底层也是ASM实现字节码处理。  
> https://www.javadoop.com/post/aspectj  
> https://javatar.iteye.com/blog/814426

##### 15. 动态代理与cglib实现的区别。  
> JDK动态代理是通过接口中的方法名，在动态生成的代理类中调用业务实现类的同名方法；  
> CGlib动态代理是通过继承业务类，生成的动态代理类是业务类的子类，通过重写业务方法进行代理；  
> https://blog.csdn.net/a837199685/article/details/68930987

##### 16. 为什么CGlib方式可以对接口实现代理。

##### 17. final的用途。
> 被final修饰的类不可以被继承、被final修饰的方法不可以被重写、被final修饰的变量不可以被改变。  
> 被final修饰的方法，JVM会尝试为之寻求内联，这对于提升Java的效率是非常重要的。因此，假如能确定方法不会被继承，那么尽量将方法定义为final的。  
> 被final修饰的常量，在准备阶段会存入调用类的常量池中。

##### 18. 写出三种单例模式实现。  
> https://www.cnblogs.com/zhaoyan001/p/6365064.html

##### 19. 如何在父类中为子类自动完成所有的hashcode和equals实现？这么做有何优劣。
> 通过反射

##### 20. 请结合OO设计理念，谈谈访问修饰符public、private、protected、default在应用设计中的作用。  
> 抽象、封装、继承、多态

##### 21. 深拷贝和浅拷贝区别。

##### 22. 数组和链表数据结构描述，各自的时间复杂度。  
> 数组利用下标定位，时间复杂度为O(1)，链表定位元素时间复杂度O(n)；  
> 数组插入或删除元素的时间复杂度O(n)，链表的时间复杂度O(1)。

##### 23. error和exception的区别，CheckedException，RuntimeException的区别。  
> Error（错误）：是程序中无法处理的错误  
> Exception（异常）：程序本身可以捕获并且可以处理的异常  
![](resources/java_foundation/1_23.png)

##### 24. 请列出5个运行时异常。  
> NullPointerException - 空指针引用异常  
> NumberFormatException - 数字格式异常  
> ClassCastException - 类型强制转换异常  
> IndexOutOfBoundsException - 下标越界异常  
> ClassNotFoundException - 无法找到指定的类异  
> IllegalArgumentException - 传递非法参数异常  
> ArithmeticException - 算术运算异常  
> ArrayStoreException - 向数组中存放与声明类型不兼容对象异常  
> NegativeArraySizeException - 创建一个大小为负数的数组错误异常  
> SecurityException - 安全异常  
> UnsupportedOperationException - 不支持的操作异常  

##### 25. 在自己的代码中，如果创建一个java.lang.String类，这个类是否可以被类加载器加载？为什么。  
> 自己写一个classLoader来加载自己写的java.lang.String类，也不会加载成功，加载java.*开头的类会抛出"java.lang.SecurityException:Prohibited package name:java.lang"，jvm的实现中已经保证了必须由bootstrp来加载。

##### 26. 说一说你对java.lang.Object对象中hashCode和equals方法的理解。在什么场景下需要重新实现这两个方法。  
> 《Effective Java》相等的对象必须具有相等的散列码（hashCode）

##### 27. 在jdk1.5中，引入了泛型，泛型的存在是用来解决什么问题。  
> 语法糖。  
> 伪泛型，强转、桥方法等。

##### 28. 这样的a.hashcode() 有什么用，与a.equals(b)有什么关系。

##### 29. 有没有可能2个不相等的对象有相同的hashcode。

##### 30. Java中的HashSet内部是如何工作的。

##### 31. 什么是序列化，怎么序列化，为什么序列化，反序列化会遇到什么问题，如何解决。

##### 32. java8的新特性。
> lambda、CompletableFuture、Time
## 二、JVM知识

##### 1. 什么情况下会发生栈内存溢出。  
> 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError异常。  
> -Xss128k:设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K（其实256K差不多了，太多用不了，而且线程多了占内存）

##### 2. JVM的内存结构，Eden和Survivor比例。
![](resources/java_foundation/2_2.jpg)
> 默认8:1

##### 3. JVM内存为什么要分成新生代，老年代，持久代。新生代中为什么要分为Eden和Survivor。
> 新生代中，每次垃圾收集时都发现大批对象死去，只有少量存活，那就选用复制算法（效率高不会产生内存碎片）。老年代因为对象存活率高，没有额外空间对它进行分配担保，那就必须使用“标记-清理”或者“标记-整理”等重量级算法来进行回收。
> 分为Eden和Survivor主要还是为了解决垃圾回收产生内存碎片。

##### 4. JVM中一次完整的GC流程是怎样的，对象如何晋升到老年代，说说你知道的几种主要的JVM参数。
> - Partial GC：并不收集整个GC堆的模式  
>     - Young GC：只收集young gen的GC
>     - Old GC：只收集old gen的GC。只有CMS的concurrent collection是这个模式
>     - Mixed GC：收集整个young gen以及部分old gen的GC。只有G1有这个模式
> - Full GC：收集整个堆，包括young gen、old gen、perm gen（如果存在的话）等所有部分的模式。
> 最简单的分代式GC策略，按HotSpot VM的serial GC的实现来看，触发条件是：  
young GC：当young gen中的eden区分配满的时候触发。注意young GC中有部分存活对象会晋升到old gen，所以young GC后old gen的占用量通常会有所升高。  
full GC：当准备要触发一次young GC时，如果发现统计数据说之前young GC的平均晋升大小比目前old gen剩余的空间大，则不会触发young GC而是转为触发full GC（因为HotSpot VM的GC里，除了CMS的concurrent collection之外，其它能收集old gen的GC都会同时收集整个GC堆，包括young gen，所以不需要事先触发一次单独的young GC）；或者，如果有perm gen的话，要在perm gen分配空间但已经没有足够空间时，也要触发一次full GC；或者System.gc()、heap dump带GC，默认也是触发full GC。

> -XX:MaxTenuringThreshold 默认值15
> -XX:TargetSurvivorRatio survivor区的目标使用率。默认50

##### 5. 你知道哪几种垃圾收集器，各自的优缺点，重点讲下cms和G1，包括原理，流程，优缺点。 垃圾回收算法的实现原理。


##### 6. 当出现了内存溢出，你怎么排错。
`
jmap -dump:live,format=b,file=/home/heap.bin pid
`
> IBM的MemoryAnalyzer或HeapAnalyzer

##### 7. JVM内存模型的相关知识了解多少，比如重排序，内存屏障，happen-before，主内存，工作内存等。
> JMM屏蔽各种硬件和操作系统的内存访问差异，主要目标是定义程序中各个变量的访问规则。  
> 所有变量都存在主内存中，线程对变量操作只能先从主存拷贝再修改。
> 原子性：原子性变量操作包括read、load、assign、use、store和write。lock、unlock对应的synchronized块之间的操作也具备原子性。  
> 可见性：当一个线程修改了共享变量的值，其他线程能够立即得知这个修改。volatile关键字，synchronized和final也可以保证。  
> 有序性：在本线程所有操作都是有序的，一个线程观察另一个线程所有操作都是无序的。  

> 指令重排：编译器优化的重排、指令并行的重排（CPU流水线）、内存系统的重排（三级缓存和内存存在同步时间差）  
> 内存屏障：插入内存屏障禁止在内存屏障前后的指令执行重排序优化  
> happen-before：程序次序规则（一个线程内顺序执行）、管程锁定（unlock先写lock）、volatile（写先于读）、线程启动|终止|中断、对象终结、传递性（A先行B，B先行C，A先行C）
https://blog.csdn.net/javazejian/article/details/72772461#%E7%90%86%E8%A7%A3java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F%E4%B8%8Ejava%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B

##### 8. 简单说说你了解的类加载器，可以打破双亲委派么，怎么打破。
> 加载：加载Class（可以从jar、war、动态代理、jsp等获取），在内存中生成代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的入口。
> 验证：确保Class文件的字节流中包含的信息符合当前虚拟机的要求，不会危害虚拟机自身安全。文件格式、元数据验证、字节码验证、符号引用验证。
> 准备：为类变量分配内存并设置类变量的初始值，即在方法区中分配变量所使用的内存空间。静态变量初始值为0，final 静态变量的初始值为实际值。
> 解析：将常量池的符号引用（Class文件中的CONSTANT_xxx_INFO）替换成直接引用。
> 初始化：将准备阶段的初始值赋值（静态成员）。

##### 9. 讲讲JAVA的反射机制。

##### 10. 你们线上应用的JVM参数有哪些。  
`
-server -Xms${2*$MEM_TOTAL/5}g -Xmx${2*$MEM_TOTAL/5}g -XX:MaxDirectMemorySize=${$MEM_TOTAL/4}g -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=512m -XX:NewRatio=2 -XX:SurvivorRatio=10 -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSMaxAbortablePrecleanTime=5000 -XX:+CMSClassUnloadingEnabled -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=80 
`

`
-XX:NewRatio
新生代（Eden + 2*S）与老年代（不包括永久区）的比值
4 表示新生代 ：老年代 = 1：4 ，意思是老年代占 4/5
`

`
-XX:SurvivorRatio
2个Survivor区和Eden区的比值
8 表示 两个Survivor ： Eden = 2： 8 ，每个Survivor占 1/10
`

##### 11. g1和cms区别,吞吐量优先和响应优先的垃圾收集器选择。

##### 12. 怎么打出线程栈信息。
```bash
jstack -l pid > *.log
top -H p pid
printf %x 线程ID
```

##### 13. 请解释如下jvm参数的含义：  
`
-server -Xms512m -Xmx512m -Xss1024K
-XX:PermSize=256m -XX:MaxPermSize=512m -
XX:MaxTenuringThreshold=20XX:CMSInitiatingOccupancyFraction=80 -
XX:+UseCMSInitiatingOccupancyOnly
`

## 开源框架知识
##### 1. 简单讲讲tomcat结构，以及其类加载器流程，线程模型等。

##### 2. tomcat如何调优，涉及哪些参数 。  
> server.xml  
> - **Connector节点**  
> protocol 默认值HTTP/1.1，Tomcat7 BIO或APR；Tomcat8 NIO或APR  
> acceptCount  (default=100)  
> maxConnections 最大链接数  (NIO default=10000 APR/native default=8192)  
> maxThreads 工作线程池 (default=200)  
> - **Executor节点**  
> maxThreads  (default=200)  
> minSpareThreads 最小线程数  (default=25)  
> threadPriority 线程优先级  

##### 3. 讲讲Spring加载流程。

##### 4. Spring AOP的实现原理。

##### 5. 讲讲Spring事务的传播属性。
> REQUIRED 如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务  
> MANDATORY 支持当前事务，如果当前没有事务，就抛出异常  
> NEVER 以非事务方式执行，如果当前存在事务，则抛出异常  
> NOT_SUPPORTED 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起  
> REQUIRES_NEW 新建事务，如果当前存在事务，把当前事务挂起  
> SUPPORTS 支持当前事务，如果当前没有事务，就以非事务方式执行  
> NESTED 支持当前事务，新增Savepoint点，与当前事务同步提交或回滚  

##### 6. Spring如何管理事务的。

##### 7. Spring怎么配置事务（具体说出一些关键的xml 元素）。

##### 8. 说说你对Spring的理解，非单例注入的原理？它的生命周期？循环注入的原理，aop的实现原理，说说aop中的几个术语，它们是怎么相互工作的。
##### 9. Springmvc 中DispatcherServlet初始化过程。
##### 10. netty的线程模型，netty如何基于reactor模型上实现的。
##### 11. 为什么选择netty。
##### 12. 什么是TCP粘包，拆包。解决方式是什么。
##### 13. netty的fashwheeltimer的用法，实现原理，是否出现过调用不够准时，怎么解决。
##### 14. netty的心跳处理在弱网下怎么办。
##### 15. netty的通讯协议是什么样的。
##### 16. springmvc用到的注解，作用是什么，原理
##### 17. springboot启动机制。

## 三、操作系统
##### 1. Linux系统下你关注过哪些内核参数，说说你知道的。
**配置文件：#vi /etc/sysctl.conf**  
net.core.somaxconn = 128  全连接队列大小  socket创建时也可以传入  
net.ipv4.tcp_max_syn_backlog = 128  半连接队列的大小  
net.ipv4.tcp_keepalive_time = 7200  tcp keepalive 防止长时间无流量，长时间占用句柄  
net.ipv4.tcp_fin_timeout = 60   FIN_WAIT_2状态的超时时长  

**配置文件：#vi /etc/security/limits.conf**  
句柄数量：  
soft　　nofile　　1024  
hard　　nofile　　1024  
##### 2. Linux下IO模型有几种，各自的含义是什么。
- 阻塞IO
- 非阻塞IO
- IO复用
- 信号驱动IO
- 异步IO

##### 3. epoll和poll有什么区别。

##### 4. 平时用到哪些Linux命令。
ps -ef  
tail  
less  
grep  -A B C  
##### 5. 用一行命令查看文件的最后五行。
head -n 5 file  
tail -n 5 file  
tail -500f file
##### 6. 用一行命令输出正在运行的java进程。
ps -ef | grep java
##### 7. 介绍下你理解的操作系统中线程切换过程。
##### 8. 进程和线程的区别。
https://blog.csdn.net/swjtuwyp/article/details/51469552 
1. 进程是操作系统分配资源的最小单位，线程是程序执行的最小单位（CPU调度线程） 
2. 进程是线程的容器，一个进程由一个或多个线程组成，线程是一个进程中代码的不同执行路径 
3. 进程之间相互独立，但同一进程下的各个线程之间共享程序的内存空间(包括代码段、数据集、堆等)及一些进程级的资源(如打开文件和信号)，某进程内的线程在其它进程不可见； 
4. 调度和切换：线程上下文切换比进程上下文切换要快得多。
##### 9. top 命令之后有哪些内容，有什么作用。
1. 平均负载：整体的负载情况，可能是CPU或者I/O忙
2. tasks
3. cpu
4. mem
5. swap
6. pid：RES、VIRT等 
##### 10. 线上CPU爆高，请问你如何找到问题所在。
```bash
jstack -l pid > *.log

top -H p pid
printf %x 线程ID
```

## 四、多线程
##### 1. 多线程的几种实现方式，什么是线程安全。
Thread、Runnable、Callable
##### 2. volatile的原理，作用，能代替锁么。
https://www.cnblogs.com/xrq730/p/7048693.html

##### 3. 画一个线程的生命周期状态图。
![](resources/java_foundation/4_1.jpg)

##### 4. sleep和wait的区别。
##### 5. sleep和sleep(0)的区别。
##### 6. Lock与Synchronized的区别 。
##### 7. synchronized的原理是什么，一般用在什么地方(比如加在静态方法和非静态方法的区别，静态方法和非静态方法同时执行的时候会有影响吗)，解释以下名词：重排序，自旋锁，偏向锁，轻量级锁，可重入锁，公平锁，非公平锁，乐观锁，悲观锁。
##### 8. 用过哪些原子类，他们的原理是什么。
##### 9. JUC下研究过哪些并发工具，讲讲原理。
##### 10. 用过线程池吗，如果用过，请说明原理，并说说newCache和newFixed有什么区别，构造函数的各个参数的含义是什么，比如coreSize，maxsize等。
##### 11. 线程池的关闭方式有几种，各自的区别是什么。
1. shutdownNow：线程池拒接收新提交的任务，同时立马关闭线程池，线程池里的任务不再执行。当我们使用shutdownNow方法关闭线程池时，一定要对任务里进行异常捕获。
2. shutdown：线程池拒接收新提交的任务，同时等待线程池里的任务执行完毕后关闭线程池。当我们使用shuwdown方法关闭线程池时，一定要确保任务里不会有永久阻塞等待的逻辑，否则线程池就关闭不了。

##### 12. 假如有一个第三方接口，有很多个线程去调用获取数据，现在规定每秒钟最多有10个线程同时调用它，如何做到。
令牌桶、漏桶。或者信号量
https://www.liangzl.com/get-article-detail-12277.html

##### 13. spring的controller是单例还是多例，怎么保证并发的安全。
##### 14. 用三个线程按顺序循环打印abc三个字母，比如abcabcabc。
##### 15. ThreadLocal用过么，用途是什么，原理是什么，用的时候要注意什么。
##### 16. 如果让你实现一个并发安全的链表，你会怎么做。
##### 17. 有哪些无锁数据结构，他们实现的原理是什么。
##### 18. 讲讲java同步机制的wait和notify。

##### 19. CAS机制是什么，如何解决ABA问题。

##### 20. 多线程如果线程挂住了怎么办。
##### 21. countdowlatch和cyclicbarrier的内部原理和用法，以及相互之间的差别(比如countdownlatch的await方法和是怎么实现的)。
##### 22. 对AbstractQueuedSynchronizer了解多少，讲讲加锁和解锁的流程，独占锁和公平所 加锁有什么不同。
##### 23. 使用synchronized修饰静态方法和非静态方法有什么区别。
##### 24. 简述ConcurrentLinkedQueue和LinkedBlockingQueue的用处和不同之处。
##### 25. 导致线程死锁的原因？怎么解除线程死锁。
1. 互斥条件
2. 请求保持
3. 不剥夺条件
4. 环路等待
##### 26. 非常多个线程（可能是不同机器），相互之间需要等待协调，才能完成某种工作，问怎么设计这种协调方案。
##### 27. 用过读写锁吗，原理是什么，一般在什么场景下用。
##### 28. 开启多个线程，如果保证顺序执行，有哪几种实现方式，或者如何保证多个线程都执行完再拿到结果。
##### 29. 延迟队列的实现方式，delayQueue和时间轮算法的异同。

##TCP与HTTP

##### 1. http1.0和http1.1有什么区别。
http1.0两个问题：
    1.连接无法复用 
    2.队首请求阻塞
http1.1 
    1.TCP通道长连接 
    2.新增首部字段Host 
    3.新增Connection默认开启Keep-Alive 
    4.新增http流水线 http pipelining可以在http端同时发起多个请求但还是回收数据还是按照响应的一一接受
http2.0
    1.Multiplexing pipelining把多个HTTP请求发送到同一个TCP连接中一一发送 在发送过程中不用等待结果
    2.服务器推送* 
    3.二进制传输
    4.头部压缩
##### 2. TCP三次握手和四次挥手的流程，为什么断开连接要4次,如果握手只有两次，会出现什么。
握手：1.B不能保证A是否接受到第二条消息就开始连接准备  2.A发送数据丢失下可能B又接受到这样就会认为再此开始连接
四次挥手
   A：FIN。进入FIN_WAIT_1状态
   B：CLOSE。进入CLOSE_WAIT状态。
   A：收到B的确认。进入FIN_WAIT_2状态。如果这个时候B直接跑路A将一直停留但Linux有tcp_fin_timeout处理
   B：不玩了。
   A：ACK回复知道了。FIN_WAIT_2状态结束。如果这时A直接跑路万一B没收到就再也收不到了且防止端口空出被占用
   A：发送之后。进入TIME_WAIT(2MSL)的状态。MSL报文最大生存时间超过这个时间报文将被丢弃
   
##### 3. TIME_WAIT和CLOSE_WAIT的区别。
TIME_WAIT：是请求端为了防止处理端没接收到最后ack而一直等待
CLOSE_WAIT：是处理端处理剩下请求时间

##### 4. 说说你知道的几种HTTP响应码，比如200, 302, 404。
301/302 重定向，404 服务端未找到资源
https://baike.baidu.com/item/HTTP%E7%8A%B6%E6%80%81%E7%A0%81/5053660?fr=aladdin

##### 5. 当你用浏览器打开一个链接的时候，计算机做了哪些工作步骤。


##### 6. TCP/IP如何保证可靠性，说说TCP头的结构。

##### 7. 如何避免浏览器缓存。
##### 8. 如何理解HTTP协议的无状态性。
##### 9. 简述Http请求get和post的区别以及数据包格式。
##### 10. HTTP有哪些method
##### 11. 简述HTTP请求的报文格式。
##### 12. HTTP的长连接是什么意思。
##### 13. HTTPS的加密方式是什么，讲讲整个加密解密流程。
##### 14. Http和https的三次握手有什么区别。
##### 15. 什么是分块传送。
##### 16. Session和cookie的区别

## 架构设计与分布式

##### 1. 用java自己实现一个LRU。
##### 2. 分布式集群下如何做到唯一序列号。
##### 2. 设计一个秒杀系统，30分钟没付款就自动关闭交易。
##### 2. 如何使用redis和zookeeper实现分布式锁？有什么区别优缺点，会有什么问题，分别适用什么场景。（延伸：如果知道redlock，讲讲他的算法实现，争议在哪里）
##### 2. 如果有人恶意创建非法连接，怎么解决。
##### 2. 分布式事务的原理，优缺点，如何使用分布式事务，2pc 3pc 的区别，解决了哪些问题，还有哪些问题没解决，如何解决，你自己项目里涉及到分布式事务是怎么处理的。
##### 2. 什么是一致性hash。
##### 2. 什么是restful，讲讲你理解的restful。
##### 2. 如何设计一个良好的API。
##### 2. 如何设计建立和保持100w的长连接。
##### 2. 解释什么是MESI协议(缓存一致性)。
##### 2. 说说你知道的几种HASH算法，简单的也可以。
##### 2. 什么是paxos算法， 什么是zab协议。
##### 2. 一个在线文档系统，文档可以被编辑，如何防止多人同时对同 一份文档进行编辑更新。
##### 2. 线上系统突然变得异常缓慢，你如何查找问题。
##### 2. 说说你平时用到的设计模式
##### 2. Dubbo的原理，有看过源码么，数据怎么流转的，怎么实现集群，负载均衡，服务注册 和发现，重试转发，快速失败的策略是怎样的 。
##### 2. 一次RPC请求的流程是什么。
##### 2. 自己实现过rpc么，原理可以简单讲讲。Rpc要解决什么问题。
##### 2. 异步模式的用途和意义。
##### 2. 编程中自己都怎么考虑一些设计原则的，比如开闭原则，以及在工作中的应用。
##### 2. 设计一个社交网站中的“私信”功能，要求高并发、可扩展等等。 画一下架构图。
##### 2. MVC模式，即常见的MVC框架。
##### 2. 聊下曾经参与设计的服务器架构并画图，谈谈遇到的问题，怎么解决的。
##### 2. 应用服务器怎么监控性能，各种方式的区别。
##### 2. 如何设计一套高并发支付方案，架构如何设计。
##### 2. 如何实现负载均衡，有哪些算法可以实现。
##### 2. Zookeeper的用途，选举的原理是什么。
##### 2. Zookeeper watch机制原理。
##### 2. Mybatis的底层实现原理。
##### 2. 请思考一个方案，实现分布式环境下的countDownLatch。
##### 2. 后台系统怎么防止请求重复提交。
##### 2. 描述一个服务从发布到被消费的详细过程。
##### 2. 讲讲你理解的服务治理。
##### 2. 如何做到接口的幂等性。
##### 2. 如何做限流策略，令牌桶和漏斗算法的使用场景。
##### 2. 什么叫数据一致性，你怎么理解数据一致性。
##### 2. 分布式服务调用方，不依赖服务提供方的话，怎么处理服务方挂掉后，大量无效资源请求的浪费，如果只是服务提供方吞吐不高的时候该怎么做，如果服务挂了，那么一会重启，该怎 么做到最小的资源浪费，流量半开的实现机制是什么。
##### 2. dubbo的泛化调用怎么实现的，如果是你，你会怎么做。
##### 2. 远程调用会有超时现象，如果做到优雅的控制，JDK自带的超时机制有哪些，怎么实现的。

## 算法

10亿个数字里里面找最小的10个。
有1亿个数字，其中有2个是重复的，快速找到它，时间和空间要最优。
2亿个随机生成的无序整数,找出中间大小的值。
给一个不知道长度的（可能很大）输入字符串，设计一种方案，将重复的字符排重。
遍历二叉树。
有3n+1个数字，其中3n个中是重复的，只有1个是不重复的，怎么找出来。
写一个字符串（如：www.javastack.cn）反转函数。
常用的排序算法，快排，归并、冒泡。 快排的最优时间复杂度，最差复杂度。冒泡排序的 优化方案。
二分查找的时间复杂度，优势。
一个已经构建好的TreeSet，怎么完成倒排序。
什么是B+树，B-树，列出实际的使用场景。
一个单向链表，删除倒数第N个数据。
200个有序的数组，每个数组里面100个元素，找出top20的元素。
单向链表，查找中间的那个元素。
## 数据库知识

数据库隔离级别有哪些，各自的含义是什么，MYSQL默认的隔离级别是是什么。
什么是幻读。
MYSQL有哪些存储引擎，各自优缺点。
高并发下，如何做到安全的修改同一行数据。
乐观锁和悲观锁是什么，INNODB的标准行级锁有哪2种，解释其含义。
SQL优化的一般步骤是什么，怎么看执行计划，如何理解其中各个字段的含义。
数据库会死锁吗，举一个死锁的例子，mysql怎么解决死锁。
MYsql的索引原理，索引的类型有哪些，如何创建合理的索引，索引如何优化。
聚集索引和非聚集索引的区别。
select for update 是什么含义，会锁表还是锁行或是其他。
为什么要用Btree实现，它是怎么分裂的，什么时候分裂，为什么是平衡的。
数据库的ACID是什么。
某个表有近千万数据，CRUD比较慢，如何优化。
Mysql怎么优化table scan的。
如何写sql能够有效的使用到复合索引。
mysql中in 和exists 区别。
数据库自增主键可能的问题。
MVCC的含义，如何实现的。
你做过的项目里遇到分库分表了吗，怎么做的，有用到中间件么，比如sharding jdbc等,他 们的原理知道么。
MYSQL的主从延迟怎么解决。

## 消息队列
https://www.cnblogs.com/rjzheng/p/8994962.html
##### 1. 消息队列的使用场景。使用了消息队列会有什么缺点
解耦、异步、削峰  
缺点：  
系统可用性降低:消息队列也会挂  
系统复杂性增加:一致性问题、如何保证消息不被重复消费，如何保证保证消息可靠传输。  

##### 2. 消息的重发，补充策略。
##### 3. 如何保证消息的有序性。
##### 4. 用过哪些MQ，和其他mq比较有什么优缺点，MQ的连接是线程安全的吗，你们公司的MQ服务架构怎样的。
https://blog.csdn.net/paincupid/article/details/79721817

||kafka|rocketmq|
| --- | --- | --- |
|实时性|短轮询|长轮询，可以保证push实时性|
|重试|不支持|消息失败定时重试|
|顺序消息|支持，但宕机会乱序|分区顺序，不会乱序|
|定时消息|不支持|可以设置定时级别|
|消息查询|不支持|可根据topic、key、最大消息数量、开始时间、结束时间查询|
|回溯消息|理论上可以按照偏移来回溯消息|按照时间来回溯消息，精度毫秒|


##### 5. MQ系统的数据如何保证不丢失。
##### 6. rabbitmq如何实现集群高可用。
##### 7. kafka吞吐量高的原因。
##### 8. kafka 和其他消息队列的区别，kafka 主从同步怎么实现。
##### 9. 利用mq怎么实现最终一致性。
##### 10. 使用kafka有没有遇到什么问题，怎么解决的。
##### 11. MQ有可能发生重复消费，如何避免，如何做到幂等。
##### 12. MQ的消息延迟了怎么处理，消息可以设置过期时间么，过期了你们一般怎么处理。

## 缓存

##### 1. 常见的缓存策略有哪些，如何做到缓存(比如redis)与DB里的数据一致性，你们项目中用到了什么缓存系统，如何设计的。
https://www.cnblogs.com/rjzheng/p/9041659.html
主从 缓存方案

##### 2. 如何防止缓存击穿和雪崩。
##### 3. 缓存数据过期后的更新如何设计。
##### 4. redis的list结构相关的操作。
##### 5. Redis的数据结构都有哪些。
##### 6. Redis的使用要注意什么，讲讲持久化方式，内存设置，集群的应用和优劣势，淘汰策略等。
https://www.cnblogs.com/kevingrace/p/5685332.html

1. volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
5. allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
6. no-enviction（驱逐）：禁止驱逐数据
##### 7. redis2和redis3的区别，redis3内部通讯机制。
##### 8. 当前redis集群有哪些玩法，各自优缺点，场景。
##### 9. Memcache的原理，哪些数据适合放在缓存中。
##### 10. redis和memcached 的内存管理的区别。
##### 11. Redis的并发竞争问题如何解决，了解Redis事务的CAS操作吗。
##### 12. Redis的选举算法和流程是怎样的。
##### 13. redis的持久化的机制，aof和rdb的区别。
##### 14. redis的集群怎么同步的数据的。
##### 15. 知道哪些redis的优化操作。
##### 16. Reids的主从复制机制原理。
##### 17. Redis的线程模型是什么。
##### 18. 请思考一个方案，设计一个可以控制缓存总体大小的自动适应的本地缓存。
##### 19. 如何看待缓存的使用（本地缓存，集中式缓存），简述本地缓存和集中式缓存和优缺点。
##### 20. 本地缓存在并发使用时的注意事项。

## 搜索

##### 1. elasticsearch了解多少，说说你们公司es的集群架构，索引数据大小，分片有多少，以及一些调优手段 。elasticsearch的倒排索引是什么。
##### 2. elasticsearch 索引数据多了怎么办，如何调优，部署。
##### 3. elasticsearch是如何实现master选举的。
##### 4. 详细描述一下Elasticsearch索引文档的过程。
##### 5. 详细描述一下Elasticsearch搜索的过程。
##### 6. Elasticsearch在部署时，对Linux的设置有哪些优化方法？
##### 7. lucence内部结构是什么。