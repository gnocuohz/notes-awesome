SocketImpl 和 SocketChannelImpl 都持有了 FileDescriptor，抽空看了一下底层有没有差别，发现其实是没有区别的。  
**以 Windows 为例最终都调用了 winsock2.h 的 WINSOCK_API_LINKAGE SOCKET WSAAPI accept(SOCKET s,struct sockaddr *addr,int *addrlen); 函数。**
#### SocketImpl
```java
ServerSocket#accept
    ServerSocket#implAccept
        AbstractPlainSocketImpl#accept
            DualStackPlainSocketImpl#socketAccept
                DualStackPlainSocketImpl#accept0
```
以 Windows 为例最终调用了 jdk/src/windows/native/java/net/DualStackPlainSocketImpl.c#Java_java_net_DualStackPlainSocketImpl_accept0
```cpp
JNIEXPORT jint JNICALL Java_java_net_DualStackPlainSocketImpl_accept0
  (JNIEnv *env, jclass clazz, jint fd, jobjectArray isaa) {
    int newfd, port=0;
    jobject isa;
    jobject ia;
    SOCKETADDRESS sa;
    int len = sizeof(sa);

    memset((char *)&sa, 0, len);
    // 关键就是这个函数
    newfd = accept(fd, (struct sockaddr *)&sa, &len);

    if (newfd == INVALID_SOCKET) {
        if (WSAGetLastError() == -2) {
            JNU_ThrowByName(env, JNU_JAVAIOPKG "InterruptedIOException",
                            "operation interrupted");
        } else {
            JNU_ThrowByName(env, JNU_JAVANETPKG "SocketException",
                            "socket closed");
        }
        return -1;
    }

    ia = NET_SockaddrToInetAddress(env, (struct sockaddr *)&sa, &port);
    isa = (*env)->NewObject(env, isa_class, isa_ctorID, ia, port);
    (*env)->SetObjectArrayElement(env, isaa, 0, isa);

    return newfd;
}
```
上面提到的 DualStackPlainSocketImpl#socketAccept 方法中保存了 Socket 的 fd 和远程地址信息
```java
void socketAccept(SocketImpl s) throws IOException {
    int nativefd = checkAndReturnNativeFD();

    if (s == null)
        throw new NullPointerException("socket is null");

    int newfd = -1;
    InetSocketAddress[] isaa = new InetSocketAddress[1];
    if (timeout <= 0) {
        newfd = accept0(nativefd, isaa);
    } else {
        configureBlocking(nativefd, false);
        try {
            waitForNewConnection(nativefd, timeout);
            newfd = accept0(nativefd, isaa);
            if (newfd != -1) {
                configureBlocking(newfd, true);
            }
        } finally {
            configureBlocking(nativefd, true);
        }
    }
    // 这两步更新保存的
    /* Update (SocketImpl)s' fd */
    fdAccess.set(s.fd, newfd);
    /* Update socketImpls remote port, address and localport */
    InetSocketAddress isa = isaa[0];
    s.port = isa.getPort();
    s.address = isa.getAddress();
    s.localport = localport;
}
```
#### ServerSocketChannelImpl
```java
ServerSocketChannelImpl#accept()
    ServerSocketChannelImpl#accept(FileDescriptor, FileDescriptor, InetSocketAddress[])
        ServerSocketChannelImpl#accept0
```
以 Windows 为例最终调用了 jdk/src/windows/native/sun/nio/ch/ServerSocketChannelImpl.c#Java_sun_nio_ch_ServerSocketChannelImpl_accept0
```cpp
JNIEXPORT jint JNICALL
Java_sun_nio_ch_ServerSocketChannelImpl_accept0(JNIEnv *env, jobject this,
                                                jobject ssfdo, jobject newfdo,
                                                jobjectArray isaa)
{
    jint ssfd = (*env)->GetIntField(env, ssfdo, fd_fdID);
    jint newfd;
    SOCKETADDRESS sa;
    jobject remote_ia;
    int remote_port;
    jobject isa;
    int addrlen = sizeof(sa);

    memset((char *)&sa, 0, sizeof(sa));
    // 关键就是这个函数
    newfd = (jint)accept(ssfd, (struct sockaddr *)&sa, &addrlen);
    if (newfd == INVALID_SOCKET) {
        int theErr = (jint)WSAGetLastError();
        if (theErr == WSAEWOULDBLOCK) {
            return IOS_UNAVAILABLE;
        }
        JNU_ThrowIOExceptionWithLastError(env, "Accept failed");
        return IOS_THROWN;
    }

    (*env)->SetIntField(env, newfdo, fd_fdID, newfd);
    remote_ia = NET_SockaddrToInetAddress(env, (struct sockaddr *)&sa, (int *)&remote_port);

    isa = (*env)->NewObject(env, isa_class, isa_ctorID,
                            remote_ia, remote_port);
    (*env)->SetObjectArrayElement(env, isaa, 0, isa);

    return 1;
}
```
SocketChannelImpl 的构造函数保存了 fd 和远程的地址信息
```java
SocketChannelImpl(SelectorProvider sp, FileDescriptor fd, InetSocketAddress remote) throws IOException
{
    super(sp);
    this.fd = fd;
    this.fdVal = IOUtil.fdVal(fd);
    this.state = ST_CONNECTED;
    this.localAddress = Net.localAddress(fd);
    this.remoteAddress = remote;
}
```