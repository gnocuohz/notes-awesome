### 一、Servlet Filter
#### 1. ApplicationFilterFactory 创建 ApplicationFilterChain，并将 Filter 添加进去
ApplicationDispatcher#invoke(ServletRequest, ServletResponse, State)  
```java
private void invoke(ServletRequest request, ServletResponse response,
        State state) throws IOException, ServletException {
    /**
    * 省略代码
    */
    
    // Get the FilterChain Here
    ApplicationFilterChain filterChain =
            ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);

    // Call the service() method for the allocated servlet instance
    try {
        // for includes/forwards
        if ((servlet != null) && (filterChain != null)) {
           filterChain.doFilter(request, response);
         }
        // Servlet Service Method is called by the FilterChain
    } catch (Exception e) {
        /**
        * 省略代码
        */
    }
    
    // Release the filter chain (if any) for this request
    try {
        if (filterChain != null)
            filterChain.release();
    } catch (Throwable e) {
        ExceptionUtils.handleThrowable(e);
        wrapper.getLogger().error(sm.getString("standardWrapper.releaseFilters",
                         wrapper.getName()), e);
        // FIXME: Exception handling needs to be similar to what is in the StandardWrapperValue
    }
    
    /**
    * 省略代码
    */
}
```
#### 2. 执行 doFilter，然后回去调用内部方法 internalDoFilter
ApplicationFilterChain#internalDoFilter(ServletRequest, ServletResponse)
```java
private void internalDoFilter(ServletRequest request, ServletResponse response)throws IOException, ServletException {
    // Call the next filter if there is one
    if (pos < n) {
        ApplicationFilterConfig filterConfig = filters[pos++];
        try {
            Filter filter = filterConfig.getFilter();

            if (request.isAsyncSupported() && "false".equalsIgnoreCase(
                    filterConfig.getFilterDef().getAsyncSupported())) {
                request.setAttribute(Globals.ASYNC_SUPPORTED_ATTR, Boolean.FALSE);
            }
            if( Globals.IS_SECURITY_ENABLED ) {
                final ServletRequest req = request;
                final ServletResponse res = response;
                Principal principal =
                    ((HttpServletRequest) req).getUserPrincipal();

                Object[] args = new Object[]{req, res, this};
                SecurityUtil.doAsPrivilege ("doFilter", filter, classType, args, principal);
            } else {
                // 关注这行
                filter.doFilter(request, response, this);
            }
        } catch (IOException | ServletException | RuntimeException e) {
            
        } 
        return;
    }

    /**
    * 省略代码
    */
}
```
#### 3. ApplicationFilterChain#pos 实例属性标志当前执行链的位置，每次调用都会++获取下一个
filter.doFilter(request, response, this); 就是用户实现的 javax.servlet.Filter#doFilter 方法，实现方法时一般会用 FilterChain.doFilter 执行下一个 Filter，也就是再去调用 ApplicationFilterChain#doFilter，依次循环调用。

### 二、SpringMVC HandlerInterceptor
#### 1. HandlerExecutionChain#interceptors 实例属性保存了所有的 HandlerInterceptor
DispatcherServlet#doDispatch(HttpServletRequest, HttpServletResponse)
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    
    /**
    * 省略代码
    */
    
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // Determine handler for the current request.
            mappedHandler = getHandler(processedRequest);

            /**
            * 省略代码
            */
            // 前置处理，HandlerInterceptor#preHandle
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // Actually invoke the handler.
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            // 后置处理，HandlerInterceptor#postHandle
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            
        }
        // 调用完成时候触发，HandlerInterceptor#afterCompletion
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        
    }
}
```
#### 2. HandlerInterceptor 的三个回调方法执行过程类似，以前置处理为例。获得 HandlerInterceptor[]，for 循环遍历，假设 HandlerInterceptor 返回 false，则执行 HandlerInterceptor#afterCompletion
HandlerExecutionChain#applyPreHandle(HttpServletRequest, HttpServletResponse)
```java
boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HandlerInterceptor[] interceptors = getInterceptors();
    if (!ObjectUtils.isEmpty(interceptors)) {
        for (int i = 0; i < interceptors.length; i++) {
            HandlerInterceptor interceptor = interceptors[i];
            if (!interceptor.preHandle(request, response, this.handler)) {
                triggerAfterCompletion(request, response, null);
                return false;
            }
            this.interceptorIndex = i;
        }
    }
    return true;
}
```

### 三、Spring AOP interception
#### 1. 以 Jdk 动态代理为例
JdkDynamicAopProxy#invoke(Object, Method, Object[])
```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    MethodInvocation invocation;
    Object oldProxy = null;
    boolean setProxyContext = false;

    TargetSource targetSource = this.advised.targetSource;
    Object target = null;

    try {
        /**
        * 省略代码
        */
        // 首先会去获取此方法的拦截链
        // Get the interception chain for this method.
        List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);

        // Check whether we have any advice. If we don't, we can fallback on direct
        // reflective invocation of the target, and avoid creating a MethodInvocation.
        if (chain.isEmpty()) {
            // We can skip creating a MethodInvocation: just invoke the target directly
            // Note that the final invoker must be an InvokerInterceptor so we know it does
            // nothing but a reflective operation on the target, and no hot swapping or fancy proxying.
            Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
            retVal = AopUtils.invokeJoinpointUsingReflection(target, method, argsToUse);
        }
        else {
            // We need to create a method invocation...
            // new 一个 ReflectiveMethodInvocation，该对象 ReflectiveMethodInvocation#currentInterceptorIndex 实例属性标志当前责任链执行的位置
            invocation = new ReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
            // Proceed to the joinpoint through the interceptor chain.
            retVal = invocation.proceed();
        }
        /**
        * 省略代码
        */
    }
}
```
#### 2. 获取第一个 MethodInterceptor 开始，每次调用都将当前对象以参数的形式传入到方法中，以 @Aspect @Around 为例都会去调用 ProceedingJoinPoint#proceed() 其实就是调用 ReflectiveMethodInvocation#proceed 方法，依次循环调用。
ReflectiveMethodInvocation#proceed
```java
public Object proceed() throws Throwable {
    //	We start with an index of -1 and increment early.
    if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
        return invokeJoinpoint();
    }
    // 从 interceptorsAndDynamicMethodMatchers List 里面获取一个 MethodInterceptor（currentInterceptorIndex 默认值 -1）
    Object interceptorOrInterceptionAdvice =
            this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
    if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
        // Evaluate dynamic method matcher here: static part will already have
        // been evaluated and found to match.
        InterceptorAndDynamicMethodMatcher dm =
                (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
        Class<?> targetClass = (this.targetClass != null ? this.targetClass : this.method.getDeclaringClass());
        if (dm.methodMatcher.matches(this.method, targetClass, this.arguments)) {
            // 关注这个方法
            return dm.interceptor.invoke(this);
        }
        else {
            // Dynamic matching failed.
            // Skip this interceptor and invoke the next in the chain.
            return proceed();
        }
    }
    else {
        // It's an interceptor, so we just invoke it: The pointcut will have
        // been evaluated statically before this object was constructed.
        return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
    }
}
```
### 四、netty ChannelPipeline
#### 1. 将 ChannelHandler 封装到 DefaultChannelHandlerContext 中，父类 AbstractChannelHandlerContext#next 和 AbstractChannelHandlerContext#prev 实例属性可以形成链式结构
DefaultChannelPipeline#addLast(EventExecutorGroup, String, ChannelHandler)
```java
public final ChannelPipeline addLast(EventExecutorGroup group, String name, ChannelHandler handler) {
    final AbstractChannelHandlerContext newCtx;
    synchronized (this) {
        checkMultiplicity(handler);
        // new DefaultChannelHandlerContext，封装加入的 ChannelHandler
        newCtx = newContext(group, filterName(name, handler), handler);
        // 添加到职责链
        addLast0(newCtx);
        
        // 下面的代码可以忽略
        if (!registered) {
            newCtx.setAddPending();
            callHandlerCallbackLater(newCtx, true);
            return this;
        }

        EventExecutor executor = newCtx.executor();
        if (!executor.inEventLoop()) {
            newCtx.setAddPending();
            executor.execute(new Runnable() {
                @Override
                public void run() {
                    callHandlerAdded0(newCtx);
                }
            });
            return this;
        }
    }
    callHandlerAdded0(newCtx);
    return this;
}
```
#### 2. 添加到职责链
DefaultChannelPipeline#addFirst0(AbstractChannelHandlerContext)
```java
private void addFirst0(AbstractChannelHandlerContext newCtx) {
    AbstractChannelHandlerContext nextCtx = head.next;
    newCtx.prev = head;
    newCtx.next = nextCtx;
    head.next = newCtx;
    nextCtx.prev = newCtx;
}
```
#### 3. 调用过程，以 ChannelInboundHandler#channelRead 为例
NioEventLoop#processSelectedKey(SelectionKey, AbstractNioChannel)
```java
private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
    final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
    
    /**
    * 省略代码
    */
    
    try {
        int readyOps = k.readyOps();
        // We first need to call finishConnect() before try to trigger a read(...) or write(...) as otherwise
        // the NIO JDK channel implementation may throw a NotYetConnectedException.
        if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
            // remove OP_CONNECT as otherwise Selector.select(..) will always return without blocking
            // See https://github.com/netty/netty/issues/924
            int ops = k.interestOps();
            ops &= ~SelectionKey.OP_CONNECT;
            k.interestOps(ops);

            unsafe.finishConnect();
        }

        // Process OP_WRITE first as we may be able to write some queued buffers and so free memory.
        if ((readyOps & SelectionKey.OP_WRITE) != 0) {
            // Call forceFlush which will also take care of clear the OP_WRITE once there is nothing left to write
            ch.unsafe().forceFlush();
        }

        // Also check for readOps of 0 to workaround possible JDK bug which may otherwise lead
        // to a spin loop
        if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
            // 咱们看这个方法吧
            unsafe.read();
        }
    } catch (CancelledKeyException ignored) {
        unsafe.close(unsafe.voidPromise());
    }
}
```
NioByteUnsafe#read 方法
```java
public final void read() {
    final ChannelConfig config = config();
    if (shouldBreakReadReady(config)) {
        clearReadPending();
        return;
    }
    // NioByteUnsafe 是 AbstractNioByteChannel 的内部类，直接调用 AbstractNioByteChannel 父类 AbstractChannel#pipeline 获取 pipeline
    final ChannelPipeline pipeline = pipeline();
    final ByteBufAllocator allocator = config.getAllocator();
    final RecvByteBufAllocator.Handle allocHandle = recvBufAllocHandle();
    allocHandle.reset(config);

    ByteBuf byteBuf = null;
    boolean close = false;
    try {
        do {
            /**
            * 省略代码，读流
            */
            
            // 看这个方法
            pipeline.fireChannelRead(byteBuf);
            byteBuf = null;
        } while (allocHandle.continueReading());

        allocHandle.readComplete();
        pipeline.fireChannelReadComplete();

        if (close) {
            closeOnRead(pipeline);
        }
    } catch (Throwable t) {
    } finally {

    }
}
```
DefaultChannelPipeline#fireChannelRead
```java
public final ChannelPipeline fireChannelRead(Object msg) {
    // 传入 HeadContext 开始调用责任链
    AbstractChannelHandlerContext.invokeChannelRead(head, msg);
    return this;
}
// 静态方法 AbstractChannelHandlerContext#invokeChannelRead(io.netty.channel.AbstractChannelHandlerContext, java.lang.Object)
static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
    final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
    EventExecutor executor = next.executor();
    if (executor.inEventLoop()) {
        // 看这个方法
        next.invokeChannelRead(m);
    } else {
        executor.execute(new Runnable() {
            @Override
            public void run() {
                next.invokeChannelRead(m);
            }
        });
    }
}
// AbstractChannelHandlerContext#invokeChannelRead(Object) 会获取当前 AbstractChannelHandlerContext 封装的 ChannelInboundHandlerAdapter（也就是初始化时添加到 pipeline 的 ChannelHandler）
private void invokeChannelRead(Object msg) {
    if (invokeHandler()) {
        try {
            // 调用时将该对象作为参数传入
            ((ChannelInboundHandler) handler()).channelRead(this, msg);
        } catch (Throwable t) {
            notifyHandlerException(t);
        }
    } else {
        fireChannelRead(msg);
    }
}
// 当用户自己实现代码时会实现
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    // 逻辑代码
    // ctx.fireChannelRead(msg); 或 super.channelRead(ctx, msg);
}
// 都会调用 AbstractChannelHandlerContext#fireChannelRead，findContextInbound() 会获取下一个符合 inbound ChannelHandler，以此循环调用。
public ChannelHandlerContext fireChannelRead(final Object msg) {
    invokeChannelRead(findContextInbound(), msg);
    return this;
}
```

### 五、dubbo Filter
#### 1. 匿名内部类方式实现，封装一个 Invoker，每次调用 filter.invoke(next, invocation) 时，将下一个需要调用的 Invoker 以方法参数的形式传入，以此链式调用。
ProtocolFilterWrapper#buildInvokerChain(final Invoker<T>, String, String)
```java
private static <T> Invoker<T> buildInvokerChain(final Invoker<T> invoker, String key, String group) {
    Invoker<T> last = invoker;
    List<Filter> filters = ExtensionLoader.getExtensionLoader(Filter.class).getActivateExtension(invoker.getUrl(), key, group);
    if (!filters.isEmpty()) {
        for (int i = filters.size() - 1; i >= 0; i--) {
            final Filter filter = filters.get(i);
            final Invoker<T> next = last;
            last = new Invoker<T>() {
                /**
                * 省略代码
                */
                @Override
                public Result invoke(Invocation invocation) throws RpcException {
                    Result asyncResult;
                    try {
                        // 关键看这一步封装
                        asyncResult = filter.invoke(next, invocation);
                    } catch (Exception e) {
                        // onError callback
                        if (filter instanceof ListenableFilter) {
                            Filter.Listener listener = ((ListenableFilter) filter).listener();
                            if (listener != null) {
                                listener.onError(e, invoker, invocation);
                            }
                        }
                        throw e;
                    }
                    return asyncResult;
                }
                /**
                * 省略代码
                */
            };
        }
    }

    return new CallbackRegistrationInvoker<>(last, filters);
}
```