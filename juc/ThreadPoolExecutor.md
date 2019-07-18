https://juejin.im/post/5d1882b1f265da1ba84aa676#comment

#### 异常处理
**线程池内部处理逻辑：**  
execute 方法直接执行 runnable.run()，如果有异常直接报错，并且创建一个新的 Worker。  
```java
final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                beforeExecute(wt, task);
                Throwable thrown = null;
                // 异常直接 throw
                try {
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        // 如果突然中止就新建一个 Worker
        processWorkerExit(w, completedAbruptly);
    }
}
```
AbstractExecutorService#submit(java.lang.Runnable) 方法会将 Runnable 转成 FutureTask。如果不 get() 就拿不到异常信息。
```java
public void run() {
    if (state != NEW ||
        !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                     null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            // 这边会 catch 异常
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```
**正确处理异常：**  
![](resources/juc/threadpool_exception.jpg)
为工作者线程设置UncaughtExceptionHandler，在uncaughtException方法中处理异常
```java
ExecutorService threadPool = Executors.newFixedThreadPool(1, r -> {
    Thread t = new Thread(r);
       t.setUncaughtExceptionHandler(
               (t1, e) -> {
                   System.out.println(t1.getName() + "线程抛出的异常"+e);
               });
       return t;
      });
threadPool.execute(()->{
   Object object = null;
   System.out.print("result## " + object.toString());
});
```
使用 ThreadPoolExecutor#afterExecute 
```java
class ExtendedExecutor extends ThreadPoolExecutor {
    // 这可是jdk文档里面给的例子。。
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        if (t == null && r instanceof Future<?>) {
            try {
                Object result = ((Future<?>) r).get();
            } catch (CancellationException ce) {
                t = ce;
            } catch (ExecutionException ee) {
                t = ee.getCause();
            } catch (InterruptedException ie) {
                Thread.currentThread().interrupt(); // ignore/reset
            }
        }
        if (t != null)
            System.out.println(t);
    }
}
```