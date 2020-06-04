很多时候我们会遇到这类场景，比如说“发生某件事情时”、“当什么产生变化时”、“如果什么状态变更时”，我们可以通过观察者模式来解耦，在领域驱动设计也称为领域事件。

下文分析 https://github.com/alibaba/COLA 的实现方式。

#### 事件总线，以 EventBus#fireAll 方法为例，该方法会根据参数 EventI 获取具体的 EventHandler 并执行。EventHub 具体实现看下面
```java
@Component
public class EventBus implements EventBusI{
    Logger logger = LoggerFactory.getLogger(EventBus.class);

    /**
     * 默认线程池
     *     如果处理器无定制线程池，则使用此默认
     */
    ExecutorService defaultExecutor = new ThreadPoolExecutor(Runtime.getRuntime().availableProcessors() + 1,
                                   Runtime.getRuntime().availableProcessors() + 1,
                                   0L,TimeUnit.MILLISECONDS,
                           new LinkedBlockingQueue<Runnable>(1000),new ThreadFactoryBuilder().setNameFormat("event-bus-pool-%d").build());

    @Autowired
    private EventHub eventHub;

    @SuppressWarnings("unchecked")
    @Override
    public Response fire(EventI event) {
        // ...省略
    }

    @Override
    public void fireAll(EventI event){
        eventHub.getEventHandler(event.getClass()).stream().map(p->{
            Response response = null;
            try {
                response = p.execute(event);
            }catch (Exception exception) {
                response = handleException(p, response, exception);
            }
            return response;
        }).collect(Collectors.toList());
    }

    @Override
    public void asyncFire(EventI event){
        // ...省略
    }

    private Response handleException(EventHandlerI handler, Response response, Exception exception) {
        // ...省略
    }
}
```
#### 事件控制中枢，EventHub#eventRepository 保存了具体 EventI 和 EventHandlerI 的对应关系
```java
@Component
public class EventHub {
    @Getter
    @Setter
    private ListMultimap<Class, EventHandlerI> eventRepository = ArrayListMultimap.create();
    
    @Getter
    private Map<Class, Class> responseRepository = new HashMap<>();
    
    public List<EventHandlerI> getEventHandler(Class eventClass) {
        List<EventHandlerI> eventHandlerIList = findHandler(eventClass);
        if (eventHandlerIList == null || eventHandlerIList.size() == 0) {
            throw new ColaException(eventClass + "is not registered in eventHub, please register first");
        }
        return eventHandlerIList;
    }

    /**
     * 注册事件
     * @param eventClz
     * @param executor
     */
    public void register(Class<? extends EventI> eventClz, EventHandlerI executor){
        eventRepository.put(eventClz, executor);
    }

    private List<EventHandlerI> findHandler(Class<? extends EventI> eventClass){
        List<EventHandlerI> eventHandlerIList = null;
        Class cls = eventClass;
        eventHandlerIList = eventRepository.get(cls);
        return eventHandlerIList;
    }

}
```
#### 接着看一下如何注册的，负责扫描在applicationContext.xml中配置的packages. 获取到CommandExecutors, intercepters, extensions, validators等交给各个注册器进行注册
```java
public class Bootstrap {
    @Getter
    @Setter
    private List<String> packages;
    private ClassPathScanHandler handler;

    @Autowired
    private RegisterFactory registerFactory;


    public void init() {
        // scanConfiguredPackages 扫描各个 packages
        Set<Class<?>> classSet = scanConfiguredPackages();
        // 注册
        registerBeans(classSet);
    }

    /**
     * @param classSet
     */
    private void registerBeans(Set<Class<?>> classSet) {
        for (Class<?> targetClz : classSet) {
            //    调用工厂方法 getRegister
            //    public RegisterI getRegister(Class<?> targetClz) {
            //        ...省略代码，和下面的获取方式相同
            //        EventHandler eventHandlerAnn = targetClz.getDeclaredAnnotation(EventHandler.class);
            //        if (eventHandlerAnn != null) {
            //            eventRegister 是通过 spring 注入的单例
            //            return eventRegister;
            //        }
            //        return null;
            //    }
            RegisterI register = registerFactory.getRegister(targetClz);
            if (null != register) {
                register.doRegistration(targetClz);
            }
        }

    }

    /**
     * Scan the packages configured in Spring xml
     *
     * @return
     */
    private Set<Class<?>> scanConfiguredPackages() {
        if (packages == null) throw new ColaException("Command packages is not specified");

        String[] pkgs = new String[packages.size()];
        handler = new ClassPathScanHandler(packages.toArray(pkgs));

        Set<Class<?>> classSet = new TreeSet<>(new ClassNameComparator());
        for (String pakName : packages) {
            classSet.addAll(handler.getPackageAllClasses(pakName, true));
        }
        return classSet;
    }
}
```
#### 以 EventRegister 为例
```java
@Component
public class EventRegister implements RegisterI {

    @Autowired
    private EventHub eventHub;

    @Override
    public void doRegistration(Class<?> targetClz) {
        // 获取 EventI （也就是上述 EventHub#eventRepository 的 key）
        Class<? extends EventI> eventClz = getEventFromExecutor(targetClz);
        EventHandlerI executor = (EventHandlerI) ApplicationContextHelper.getBean(targetClz);
        eventHub.register(eventClz, executor);
    }
    
    /**
    * 获取方式：扫描 EventHandlerI 所有方法，判断是否是 execute 方法，如果是则获取该方法的入参作为 EventHub#eventRepository 的 key
    * @param eventExecutorClz
    * @return 
    */
    private Class<? extends EventI> getEventFromExecutor(Class<?> eventExecutorClz) {
        Method[] methods = eventExecutorClz.getDeclaredMethods();
        for (Method method : methods) {
            if (isExecuteMethod(method)){
                return checkAndGetEventParamType(method);
            }
        }
        throw new ColaException("Event param in " + eventExecutorClz + " " + ColaConstant.EXE_METHOD
                                 + "() is not detected");
    }
    
    // ...省略代码
}
```