private final transient ReentrantLock lock = new ReentrantLock();
private final Condition available = lock.newCondition();
// 优先级队列
private final PriorityQueue<E> q = new PriorityQueue<E>();
添加时如果优先级队列第一个就是本身就signal
```java
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // 先添加到优先级队列
        q.offer(e);
        if (q.peek() == e) {
            leader = null;
            // 然后signal，假设上次正在take的Priority小于本次添加的，会被唤醒获取本次Delay对象再await
            available.signal();
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```
signal时如果正在available.awaitNanos(delay)，就会重新peek优先级队列第一个对象
```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null)
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return q.poll();
                first = null; // don't retain ref while waiting
                if (leader != null)
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
```