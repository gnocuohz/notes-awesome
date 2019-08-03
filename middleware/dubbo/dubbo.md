### invoker 调用链
![](resources/middleware/dubbo/dubbo_invoker.jpg)

### provider 
#### export
```java
ServiceBean#onApplicationEvent
    ServiceConfig#doExport
        ServiceConfig#doExportUrls
            ServiceConfig#doExportUrlsFor1Protocol
                // 生成代理类
                Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
                DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
                Exporter<?> exporter = protocol.export(wrapperInvoker);
                    
                    RegistryProtocol#export
                        RegistryProtocol#doLocalExport
                            
                            ProtocolFilterWrapper#export
                                ProtocolListenerWrapper#export
                                    DubboProtocol#export
                                        DubboProtocol#openServer
                                            DubboProtocol#createServer
                                            
                                                Exchangers#bind
                                                    HeaderExchanger#bind
                                                        Transporters#bind
                                                            NettyTransporter#bind
```
#### 调用链
```java
DubboProtocol#createServer
    Exchangers#bind(URL, ExchangeHandler DubboProtocol#requestHandler)
        public ExchangeServer bind(URL url, ExchangeHandler handler) throws RemotingException {
            return new HeaderExchangeServer(Transporters.bind(url, new DecodeHandler(new HeaderExchangeHandler(handler))));
        }
// 上面生成了
// new DecodeHandler(new HeaderExchangeHandler(DubboProtocol#requestHandler))

NettyServer#NettyServer(URL url, ChannelHandler handler)
    protected ChannelHandler wrapInternal(ChannelHandler handler, URL url) {
            return new MultiMessageHandler(new HeartbeatHandler(ExtensionLoader.getExtensionLoader(Dispatcher.class)
                    .getAdaptiveExtension().dispatch(handler, url)));
// 然后生成了
// new MultiMessageHandler(new HeartbeatHandler(new AllChannelHandler(new DecodeHandler(new HeaderExchangeHandler(DubboProtocol#requestHandler)), url))

// 所以netty收到消息之后调用链
MultiMessageHandler
    HeartbeatHandler
        AllChannelHandler
            DecodeHandler
                HeaderExchangeHandler
                    DubboProtocol#requestHandler
```
![](resources/middleware/dubbo/dubbo_provider_invoker_filter.png)   

### SPI
##### wrapper
ExtensionLoader#createExtension  
ExtensionLoader#loadResource 时被判断为 wrapper 的实现类，会包装实际的SPI扩展。
![](resources/middleware/dubbo/wrapper_class.jpg)

### router
### load balance
