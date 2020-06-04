#### 基础数据结构
1. string (字符串)
2. list (列表)
3. set (集合)
4. zset (有序集合)
5. hash (哈希)

#### 分布式锁
```py
# 获取锁
tag = random.nextint()  # 随机数
if redis.set(key, tag, nx=True, ex=5):
    do_something()
    redis.delifequals(key, tag)  # 假想的 delifequals 指令

# delifequals，删除锁
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

##### 可重入锁
```java
public class RedisWithReentrantLock {

  private ThreadLocal<Map<String, Integer>> lockers = new ThreadLocal<>();

  private Jedis jedis;

  public RedisWithReentrantLock(Jedis jedis) {
    this.jedis = jedis;
  }

  private boolean _lock(String key) {
    return jedis.set(key, "", "nx", "ex", 5L) != null;
  }

  private void _unlock(String key) {
    jedis.del(key);
  }

  private Map<String, Integer> currentLockers() {
    Map<String, Integer> refs = lockers.get();
    if (refs != null) {
      return refs;
    }
    lockers.set(new HashMap<>());
    return lockers.get();
  }

  public boolean lock(String key) {
    Map<String, Integer> refs = currentLockers();
    Integer refCnt = refs.get(key);
    if (refCnt != null) {
      refs.put(key, refCnt + 1);
      return true;
    }
    boolean ok = this._lock(key);
    if (!ok) {
      return false;
    }
    refs.put(key, 1);
    return true;
  }

  public boolean unlock(String key) {
    Map<String, Integer> refs = currentLockers();
    Integer refCnt = refs.get(key);
    if (refCnt == null) {
      return false;
    }
    refCnt -= 1;
    if (refCnt > 0) {
      refs.put(key, refCnt);
    } else {
      refs.remove(key);
      this._unlock(key);
    }
    return true;
  }

  public static void main(String[] args) {
    Jedis jedis = new Jedis();
    RedisWithReentrantLock redis = new RedisWithReentrantLock(jedis);
    System.out.println(redis.lock("codehole"));
    System.out.println(redis.lock("codehole"));
    System.out.println(redis.unlock("codehole"));
    System.out.println(redis.unlock("codehole"));
  }

}
```

[存结构体信息到底该使用 hash 还是 string](https://stackoverflow.com/questions/16375188/redis-strings-vs-redis-hashes-to-represent-json-efficiency)


#### pipeline、Transaction对比
 pipeline关注的是RTT时间，而transaction关注的是一致性
 -- pipeline: 一次请求，服务端顺序执行，一次返回。
 -- transaction: 多次请求（MULTI命令＋其他n个命令＋EXEC命令，所以至少是2次请求），服务端顺序执行，一次返回。
 -- 集群模式下，使用pipeline时，slot必须是对的，不然服务端会返回redirecred to slot xxx的错误；不建议使用Transaction，因为假设一个Transaction中的命令即在Master A上执行，也在Master B执行，A成功了，B因为某种原因失败了，这样数据就不一致了，这个有点类似于分布式事务，无法保证绝对一致性。

