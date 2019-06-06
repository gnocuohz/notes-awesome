实现 AQS 共享锁  
new CountDownLatch(int count) 会setState(count)  
CountDownLatch#await() 会去获取共享锁，如果count!=0 就 LockSupport.park(this)  
CountDownLatch#countDown 会将state-1，如果state=0，就释放共享锁
