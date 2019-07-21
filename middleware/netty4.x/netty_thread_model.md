Netty 线程池的类比较多，首先感受一下两大类 EventExecutor 和 EventExecutorGroup 的类继承结构。
#### EventExecutor 
比较重要的就是两类
1. SingleThreadEventLoop 派生出了一系列的 EventLoop，常用的就是NioEventLoop，每个SingleThreadEventLoop 都是一个单独的线程
2. DefaultEventExecutor 也是单独的线程，以串行方式执行提交到 LinkedBlockingQueue 所有任务。
![](resources/middleware/netty/netty_executor.png)
#### EventExecutorGroup 
与上述 EventExecutor 对应两类
1. MultithreadEventLoopGroup，主要常用的就是 NioEventLoopGroup，线程池组，用来管理 NioEventLoop
2. DefaultEventExecutorGroup，类似的用来管理 DefaultEventExecutor 
![](resources/middleware/netty/netty_executor_group.png)
### 一、初始化
####  NioEventLoopGroup
以 NioEventLoopGroup 为例
```java
EventLoopGroup bossGroup = new NioEventLoopGroup(1, new DefaultThreadFactory("boss", true));
```
调用构造方法，构造方法重载较多，下面贴一下最全的构造方法
```java
public NioEventLoopGroup(int nThreads, Executor executor, EventExecutorChooserFactory chooserFactory,
                             final SelectorProvider selectorProvider,
                             final SelectStrategyFactory selectStrategyFactory,
                             final RejectedExecutionHandler rejectedExecutionHandler) {
        super(nThreads, executor, chooserFactory, selectorProvider, selectStrategyFactory, rejectedExecutionHandler);
    }
```
1. int nThreads：初始化 NioEventLoop 数量
2. Executor executor：线程的执行器，默认 ThreadPerTaskExecutor，给 NioEventLoop 创建线程并执行
3. EventExecutorChooserFactory chooserFactory：NioEventLoop 选择器，比如 bossEventLoopGroup 接收到新的 channel，会选择一个 NioEventLoop 去注册事件
4. final SelectorProvider selectorProvider：用来实例化 Selector，每个 NioEventLoop 都有一个 Selector 实例
5. final SelectStrategyFactory selectStrategyFactory：nio select 控制策略，也就是NioEventLoop#run 循环的策略
6. final RejectedExecutionHandler rejectedExecutionHandler：类似线程池的拒绝策略

最终会调用父类 MultithreadEventExecutorGroup 的构造方法
```java
protected MultithreadEventExecutorGroup(int nThreads, Executor executor,
                                            EventExecutorChooserFactory chooserFactory, Object... args) {
    if (nThreads <= 0) {
        throw new IllegalArgumentException(String.format("nThreads: %d (expected: > 0)", nThreads));
    }
	// 上面提到的 2
    if (executor == null) {
        executor = new ThreadPerTaskExecutor(newDefaultThreadFactory());
    }
	// 创建 nThreads 数量的 EventExecutor 数组，通俗讲就是初始化这么多个线程
    children = new EventExecutor[nThreads];

    for (int i = 0; i < nThreads; i ++) {
        boolean success = false;
        try {
        	// 实例化
        	// NioEventLoopGroup#newChild 会创建 NioEventLoop
        	// DefaultEventExecutorGroup#newChild 会创建 DefaultEventExecutor
            children[i] = newChild(executor, args);
            success = true;
        } catch (Exception e) {
            // TODO: Think about if this is a good exception type
            throw new IllegalStateException("failed to create a child event loop", e);
        } finally {
        	// 创建失败的处理逻辑
            if (!success) {
                for (int j = 0; j < i; j ++) {
                    children[j].shutdownGracefully();
                }

                for (int j = 0; j < i; j ++) {
                    EventExecutor e = children[j];
                    try {
                        while (!e.isTerminated()) {
                            e.awaitTermination(Integer.MAX_VALUE, TimeUnit.SECONDS);
                        }
                    } catch (InterruptedException interrupted) {
                        // Let the caller handle the interruption.
                        Thread.currentThread().interrupt();
                        break;
                    }
                }
            }
        }
    }
	// 上面说的 3
    chooser = chooserFactory.newChooser(children);
	// 给每个线程添加 termination 事件的监听器
    final FutureListener<Object> terminationListener = new FutureListener<Object>() {
        @Override
        public void operationComplete(Future<Object> future) throws Exception {
            if (terminatedChildren.incrementAndGet() == children.length) {
                terminationFuture.setSuccess(null);
            }
        }
    };

    for (EventExecutor e: children) {
        e.terminationFuture().addListener(terminationListener);
    }
	// 缓存一下可读的副本，可以使用迭代器遍历
    Set<EventExecutor> childrenSet = new LinkedHashSet<EventExecutor>(children.length);
    Collections.addAll(childrenSet, children);
    readonlyChildren = Collections.unmodifiableSet(childrenSet);
}
```
####  NioEventLoop
以 NioEventLoop 为例
NioEventLoopGroup 初始化时会调用 newChild 初始化每个 NioEventLoop
```java
protected EventLoop newChild(Executor executor, Object... args) throws Exception {
    return new NioEventLoop(this, executor, (SelectorProvider) args[0],
        ((SelectStrategyFactory) args[1]).newSelectStrategy(), (RejectedExecutionHandler) args[2]);
}
// 也就是调用构造方法
NioEventLoop(NioEventLoopGroup parent, Executor executor, SelectorProvider selectorProvider,
                 SelectStrategy strategy, RejectedExecutionHandler rejectedExecutionHandler) {
    super(parent, executor, false, DEFAULT_MAX_PENDING_TASKS, rejectedExecutionHandler);
    if (selectorProvider == null) {
        throw new NullPointerException("selectorProvider");
    }
    if (strategy == null) {
        throw new NullPointerException("selectStrategy");
    }
    provider = selectorProvider;
    // 每个 NioEventLoop 都会去创建一个 Selector
    final SelectorTuple selectorTuple = openSelector();
    selector = selectorTuple.selector;
    unwrappedSelector = selectorTuple.unwrappedSelector;
    selectStrategy = strategy;
}
```
这些参数在 NioEventLoopGroup 提到过了，可以看到 NioEventLoopGroup 是 NioEventLoop 的 parent
#### ServerBootstrap#group 和 AbstractBootstrap#bind
以 ServerBootstrap 为例
- ServerBootstrap#group(EventLoopGroup, EventLoopGroup) 方法将 bossEventLoopGroup 和 workEventLoopGroup 初始化到自己的上下文里面
- AbstractBootstrap#bind(java.net.SocketAddress)
```java
// AbstractBootstrap#bind(SocketAddress)
//     AbstractBootstrap#doBind
//         AbstractBootstrap#initAndRegister
//             AbstractBootstrap#init （最终调用 ServerBootstrap#init 的实现）
// 入参 NioServerSocketChannel（包装了 JDK ServerSocketChannel）
void init(Channel channel) throws Exception {
    // 设置 Server Channel 配置和属性
    final Map<ChannelOption<?>, Object> options = options0();
    synchronized (options) {
        setChannelOptions(channel, options, logger);
    }

    final Map<AttributeKey<?>, Object> attrs = attrs0();
    synchronized (attrs) {
        for (Entry<AttributeKey<?>, Object> e: attrs.entrySet()) {
            @SuppressWarnings("unchecked")
            AttributeKey<Object> key = (AttributeKey<Object>) e.getKey();
            channel.attr(key).set(e.getValue());
        }
    }
    // 获取 Server Channel 的 pipeline
    ChannelPipeline p = channel.pipeline();
    
    // 获取 Child 的配置属性，给 channel 使用
    // bootstrap.group(bossGroup, workerGroup)
    // .channel(NioServerSocketChannel.class)
    // .childOption()
    // .childHandler(new ChannelInitializer<NioSocketChannel>() {});
    final EventLoopGroup currentChildGroup = childGroup;
    final ChannelHandler currentChildHandler = childHandler;
    final Entry<ChannelOption<?>, Object>[] currentChildOptions;
    final Entry<AttributeKey<?>, Object>[] currentChildAttrs;
    synchronized (childOptions) {
        currentChildOptions = childOptions.entrySet().toArray(newOptionArray(childOptions.size()));
    }
    synchronized (childAttrs) {
        currentChildAttrs = childAttrs.entrySet().toArray(newAttrArray(childAttrs.size()));
    }
    // 往 pipeline 添加初始化 Server Channel 的 ChannelInitializer（也是一个 ChannelHandler，和其他不一样的是执行完后会 remove）
    p.addLast(new ChannelInitializer<Channel>() {
        // EventLoopGroup#register(Channel) 会调用 initChannel 方法，执行完毕后会 remove 当前 ChannelInitializer
        // private boolean initChannel(ChannelHandlerContext ctx) throws Exception {
        //           if (initMap.putIfAbsent(ctx, Boolean.TRUE) == null) { // Guard against re-entrance.
        //               try {
        //                   initChannel((C) ctx.channel());
        //               } catch (Throwable cause) {
        //                   // Explicitly call exceptionCaught(...) as we removed the handler before calling initChannel(...).
        //                   // We do so to prevent multiple calls to initChannel(...).
        //                   exceptionCaught(ctx, cause);
        //               } finally {
        //                   remove(ctx);
        //               }
        //               return true;
        //           }
        //           return false;
        //       }
        @Override
        public void initChannel(final Channel ch) throws Exception {
            // 真正的初始化方法
            final ChannelPipeline pipeline = ch.pipeline();
            ChannelHandler handler = config.handler();
            if (handler != null) {
                pipeline.addLast(handler);
            }

            ch.eventLoop().execute(new Runnable() {
                @Override
                public void run() {
                    // Acceptor，处理接受到的 SocketChannel，在下面执行流程中会说明
                    pipeline.addLast(new ServerBootstrapAcceptor(
                            ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                }
            });
        }
    });
}
```

### 二、执行流程
以 ServerBootstrap 为例  
当 Selector 触发 accept 事件会调用 NioMessageUnsafe#read  
```java
private final class NioMessageUnsafe extends AbstractNioUnsafe {

    private final List<Object> readBuf = new ArrayList<Object>();

    @Override
    public void read() {
        assert eventLoop().inEventLoop();
        final ChannelConfig config = config();
        final ChannelPipeline pipeline = pipeline();
        final RecvByteBufAllocator.Handle allocHandle = unsafe().recvBufAllocHandle();
        allocHandle.reset(config);

        boolean closed = false;
        Throwable exception = null;
        try {
            try {
                do {
                    // 接受 SocketChannel
                    int localRead = doReadMessages(readBuf);
                    if (localRead == 0) {
                        break;
                    }
                    if (localRead < 0) {
                        closed = true;
                        break;
                    }

                    allocHandle.incMessagesRead(localRead);
                } while (allocHandle.continueReading());
            } catch (Throwable t) {
                exception = t;
            }

            int size = readBuf.size();
            for (int i = 0; i < size; i ++) {
                readPending = false;
                // 触发读事件，主要看这个，会去调用上面提到的 ServerBootstrapAcceptor
                pipeline.fireChannelRead(readBuf.get(i));
            }
            readBuf.clear();
            // 其他事件，可忽略
            allocHandle.readComplete();
            pipeline.fireChannelReadComplete();

            if (exception != null) {
                closed = closeOnReadError(exception);

                pipeline.fireExceptionCaught(exception);
            }

            if (closed) {
                inputShutdown = true;
                if (isOpen()) {
                    close(voidPromise());
                }
            }
        } finally {
            if (!readPending && !config.isAutoRead()) {
                removeReadOp();
            }
        }
    }
}
```
ServerBootstrap.ServerBootstrapAcceptor#channelRead，初始化 SocketChannel，初始化的参数就是构造 ServerBootstrapAcceptor 传入的 child 参数，也就是 ServerBootstrap build 链中 ServerBootstrap#childHandler 传入的 ChannelInitializer
```java
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    final Channel child = (Channel) msg;
    // 在上面也提到过，就是初始化 ServerBootstrap 传入的参数
    // bootstrap.group(bossGroup, workerGroup)
    // .channel(NioServerSocketChannel.class)
    // .childOption()
    // .childHandler(new ChannelInitializer<NioSocketChannel>() {});
    child.pipeline().addLast(childHandler);

    setChannelOptions(child, childOptions, logger);

    for (Entry<AttributeKey<?>, Object> e: childAttrs) {
        child.attr((AttributeKey<Object>) e.getKey()).set(e.getValue());
    }

    try {
        // childGroup 是 bootstrap.group(bossGroup, workerGroup) 传入的 workerGroup
        childGroup.register(child).addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                if (!future.isSuccess()) {
                    forceClose(child, future.cause());
                }
            }
        });
    } catch (Throwable t) {
        forceClose(child, t);
    }
}
```
MultithreadEventLoopGroup#register(io.netty.channel.Channel)
```java
public ChannelFuture register(Channel channel) {
    // next() 获取其中一个 worker NioEventLoop，然后注册
    return next().register(channel);
}
```
也就是说 
1. NioServerSocketChannel 注册到了 bossNioEventLoopGroup 中的一个 boss NioEventLoop
2. boss NioEventLoop 接收到 accept 事件，会接受 SocketChannel 并去调用 NioServerSocketChannel pipeline
3. NioServerSocketChannel pipeline 中 ServerBootstrapAcceptor 的会去处理每个 SocketChannel，因为 ServerBootstrapAcceptor 中持有 workerGroup，childHandler等属性，所以可以分发到 workerGroup 其中一个 worker NioEventLoop
### 三、异步优化
#### ChannelPipeline#addLast(EventExecutorGroup, String, ChannelHandler)
ChannelPipeline#addLast(EventExecutorGroup, String, ChannelHandler) 重载方法，可以在添加 ChannelHandler 时指定一个 EventExecutorGroup，也就是文章开头讲的 DefaultEventExecutorGroup
```java
ServerBootstrap childHandler =
    this.serverBootstrap.group(this.eventLoopGroupBoss, this.eventLoopGroupSelector)
        .channel(useEpoll() ? EpollServerSocketChannel.class : NioServerSocketChannel.class)
        .option(ChannelOption.SO_BACKLOG, 1024)
        .option(ChannelOption.SO_REUSEADDR, true)
        .option(ChannelOption.SO_KEEPALIVE, false)
        .childOption(ChannelOption.TCP_NODELAY, true)
        .childOption(ChannelOption.SO_SNDBUF, nettyServerConfig.getServerSocketSndBufSize())
        .childOption(ChannelOption.SO_RCVBUF, nettyServerConfig.getServerSocketRcvBufSize())
        .localAddress(new InetSocketAddress(this.nettyServerConfig.getListenPort()))
        .childHandler(new ChannelInitializer<SocketChannel>() {
            @Override
            public void initChannel(SocketChannel ch) throws Exception {
                // 这几个 addLast 方法传入了 defaultEventExecutorGroup
                ch.pipeline()
                    .addLast(defaultEventExecutorGroup, HANDSHAKE_HANDLER_NAME,
                        new HandshakeHandler(TlsSystemConfig.tlsMode))
                    .addLast(defaultEventExecutorGroup,
                        new NettyEncoder(),
                        new NettyDecoder(),
                        new IdleStateHandler(0, 0, nettyServerConfig.getServerChannelMaxIdleTimeSeconds()),
                        new NettyConnectManageHandler(),
                        new NettyServerHandler()
                    );
            }
        });
```
重载传入了 EventExecutorGroup 就会调用 childExecutor 方法，获取一个 EventExecutor
```java
private AbstractChannelHandlerContext newContext(EventExecutorGroup group, String name, ChannelHandler handler) {
    return new DefaultChannelHandlerContext(this, childExecutor(group), name, handler);
}
private EventExecutor childExecutor(EventExecutorGroup group) {
    if (group == null) {
        return null;
    }
    Boolean pinEventExecutor = channel.config().getOption(ChannelOption.SINGLE_EVENTEXECUTOR_PER_GROUP);
    if (pinEventExecutor != null && !pinEventExecutor) {
        return group.next();
    }
    Map<EventExecutorGroup, EventExecutor> childExecutors = this.childExecutors;
    if (childExecutors == null) {
        // Use size of 4 as most people only use one extra EventExecutor.
        childExecutors = this.childExecutors = new IdentityHashMap<EventExecutorGroup, EventExecutor>(4);
    }
    // Pin one of the child executors once and remember it so that the same child executor
    // is used to fire events for the same channel.
    // 获取一个 EventExecutor
    EventExecutor childExecutor = childExecutors.get(group);
    if (childExecutor == null) {
        childExecutor = group.next();
        childExecutors.put(group, childExecutor);
    }
    return childExecutor;
}
```
这样做的好处在触发 AbstractChannelHandlerContext 一系列的 invoke 方法时可以异步执行
```java
static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
    final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
    // 获取当前 ChannelHandler 执行的 EventExecutor
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        next.invokeChannelRead(m);
    } else {
        // 因为触发的是 IO 线程，next.executor() 是执行 ChannelHandler 的线程，所以会走这边
        executor.execute(new Runnable() {
            @Override
            public void run() {
                next.invokeChannelRead(m);
            }
        });
    }
}
```
#### ChannelHandler 创建线程异步
这个比较简单不举例了，一般就是添加的最后一个 ChannelHandler 创建一个新线程去执行
#### RocketMQ线程模型
![](resources/middleware/netty/rocketmq_thread_model.jpg)
RocketMQ 创建了 reactor_boss、reactor_work、netty_worker、业务线程池四个线程池解决阻塞问题。
