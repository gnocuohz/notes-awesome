## 一、知识点

## 二、常见参数配置
#### transaction-isolation：  
推荐配置为READ-COMMITTED。
#### binlog_format参数
|format|定义|优点|缺点|
| --- | --- | --- | --- |
|statement|记录的是修改SQL语句|日志文件小，节约IO，提高性能|准确性差，对一些系统函数不能准确复制或不能复制，如now()、uuid()等|
|row(推荐)|记录的是每行实际数据的变更，记两条，更新前和更新后|准确性强，能准确复制数据的变更|日志文件大，较大的网络IO和磁盘IO|
|mixed|statement和row模式的混合|准确性强，文件大小适中|有可能发生主从不一致问题|

#### sync_binlog参数
0：当事务提交后，Mysql仅仅是将binlog_cache中的数据写入binlog文件，但不执行fsync之类的磁盘 同步指令通知文件系统将缓存刷新到磁盘，而让Filesystem自行决定什么时候来做同步，这个是性能最好的。  
n：在进行n次事务提交以后，Mysql将执行一次fsync之类的磁盘同步指令，同志文件系统将Binlog文件缓存刷新到磁盘。

#### innodb_flush_log_at_trx_commit参数
0：log buffer将每秒一次地写入log file中，并且log file的flush(刷到磁盘)操作同时进行。该模式下在事务提交的时候，不会主动触发写入磁盘的操作。  
1：每次事务提交时MySQL都会把log buffer的数据写入log file，并且flush(刷到磁盘)中去，该模式为系统默认。  
2：每次事务提交时MySQL都会把log buffer的数据写入log file，但是flush(刷到磁盘)操作并不会同时进行。该模式下，MySQL会每秒执行一次 flush(刷到磁盘)操作。

#### innodb_lock_wait_timeout
死锁超时时间，默认值50s。缺点：如果设置时间太短但容易把长时间锁等待释放掉。

#### innodb_deadlock_detect
发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。缺点：假设1000个线程更新同一行，则死锁检测要执行100万次。

#### innodb_file_per_table
OFF 存在共享表空间里，也就是跟数据字典放在一起；  
ON 单独的文件，每个innodb表数据存储在以.ibd为后缀的文件中。

#### tmp_table_size
内存临时表的大小，默认是 16M。如果内存不够则使用磁盘临时表。

## 三、开发时需要注意的
#### 1. 什么时候需要RR，一般都为RC
做数据校对，例如判断上个月的余额和当前余额的差额，是否与本月的账单明细一致。希望在校对过程中，即使有用户发生了一笔新的交易，也不影响校对结果。
#### 2. 如何安全地给小表加字段？
查看information_schema 库的 innodb_trx 表中的当前事务，等待事务结束或者 kill 该事务。（另外MariaDB支持DDL NOWAIT/WAIT n 语法避免长时间等待导致业务不可用）
#### 3. 从性能和存储空间方面考量，推荐使用自增主键
自增主键的插入数据模式，都是追加操作，都不涉及到挪动其他记录，也不会触发叶子节点的分裂。并且自增主键在非主键索引占用的空间最小。
#### 4. 如何安排联合索引顺序
假设a、b两个字段都需要索引，a字段存储空间比b字段大，则建议建（a，b）和 b 两个索引。
假设有PRIMARY KEY（a，b）和KEY  c，则不需要建（c，a）索引，可以建（c，b）索引。
#### 5. 避免长事务
#### 6. 如果事务中需要锁多个行，要把最可能造成锁冲突、最可能影响并发度的锁尽量往后放
#### 7. 怎么解决由热点行更新导致的性能问题
1. 如果能确保这个业务一定不会出现死锁，可以临时把死锁检测关掉
2. 控制并发度，降低死锁检测
3. 将一行改为多行，比如把余额分成10个子余额，但这样需要考虑扣钱问题
#### 8. 怎么删除表的前 10000 行
在一个连接中循环执行 20 次 delete from T limit 500，避免长事务（delete from T limit 10000），避免多线程（20 个连接中同时执行 delete from T limit 500）
#### 9. 从性能的角度考虑，选择唯一索引还是普通索引呢
尽量选择普通索引，因为当更新记录的目标页不在内存中时，唯一索引需要将数据页读入内存，判断到没有冲突，插入这个值，语句执行结束；而普通索引来说，则是将更新记录在change buffer，语句执行就结束了。但是如果业务不能保证重复，就需要唯一索引保证。
#### 10. MySQL有时候会选错索引
平常不断地删除历史数据和新增数据的场景，mysql有可能会选错索引。sql太慢就用explain看看，有可能就是索引选错了。
#### 11. 怎么给字符串字段加索引
直接使用字符串建索引有时候可能效率较低，存储空间较大
1. 使用前缀索引
2. 例如身份证等前面相似度较大的字符串，可以采用倒序存储
3. 建另外的字段（如hash字段）建索引
#### 12. MySQL偶尔执行很慢
偶尔慢一下的那个瞬间，可能在刷脏页（flush）。innodb的redo log写满、buffer pool内存不足等情况。
合理地设置 innodb_io_capacity 的值，平时要多关注脏页比例，不要让它经常接近 75%。
```sql
select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
select @a/@b;
```
#### 13. 为什么表数据删掉一半，表文件大小不变
删除某行，innodb只会标记删除。如果之后在该行范围内插入新数据就会复用。假如表本身没有多少空洞，重建索引可能会使表文件变大。
**重建主键索引**
```sql
alter table T engine=InnoDB;
```
不推荐drop，再add。并且不论是删除主键还是创建主键，都会将整个表重建。

#### 14. count(*)慢怎么办
Innodb需要一行一行读出来累积计数，MyISAM 引擎保存总行数，所以count很快。
1. 用缓存系统保存计数
2. 在数据库保存计数

**不同的 count 用法效率**
count(字段)<count(主键 id)<count(1)≈count(\*)，所以建议尽量使用 count(\*)

#### 15. order by
Extra中"Using filesort"表示排序，mysql会给每个线程分配一个块内存（sort_buffer）用来排序。
假设从某个索引上取出来的行天然按照递增排序，就不需要再进行排序了。但维护索引是有代价的，所以需要权衡。
#### 16. 避免条件字段函数操作、隐式类型转换、隐式字符编码转换
对索引字段做函数操作，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能。
#### 17. 为什么只查一行的SQL也执行这么慢
1. 查询长时间不返回：等MDL锁；等flush；等行锁
2. 查询慢：扫描行数多；其他长事务导致undo log快照过多
#### 18. 覆盖索引不给主键索引加锁，所以更新主键索引（没有建索引的列）不更新覆盖索引的情况不会等待。也就是只锁被访问到的对象
#### 19. 在删除数据时，尽量加limit
limit删除数据时，只会扫描limit行数，不会继续扫描，所以加锁粒度更小。
#### 20. 避免字段函数操作、避免隐式转换、隐式字符编码转换
#### 21. 查询一行很慢
1. 查询长时间不返回：等MDL锁、等磁盘flush、等行锁
2. 查询慢：没走索引、长事务导致快照过多

#### 22. 主备
##### binlog 的三种格式
statement 格式：由于没有记录行信息，删除时如果主备走的索引不一致会删除不同的行  
row 格式：记录的行信息  
mixed 格式：由于可能以statement格式记录，所以也会主备不一致  
##### 实际生产上使用比较多的是双 M 结构
循环复制：规定两个库的 server id 必须不同，每个库收到主库发来的日志，判断server id是否和自己相同，相同直接丢弃日志。
##### 主备延迟
在备库执行show slave status 命令，seconds_behind_master显示了当前备库延迟，精度秒。
##### 双主切换
1. 判断备库B现在的seconds_behind_master，如果小于某个值（比如 5 秒）继续下一步，否则持续重试这一步；
2. 把主库A改成只读状态，即把readonly 设置成true；
3. 判断备库 B 的 seconds_behind_master的值，直到这个值变成 0 为止；
4. 把备库 B 改成可读写状态，也就是把 readonly 设置为 false；
5. 把业务请求切到备库 B。

上述过程会有一段时间不可用，假设4、5步互换会导致不一致。  
binlog_format=mixed时，可能有两行不一致，binlog_format=row时会有一行不一致。  
**可靠性异常切换**  
假设主库掉电，必须等到备库B seconds_behind_master=0 之后，才能切换。
#### 23. 读写分离
1. 实时请求强制走主库方案
2. sleep几秒（不推荐）
3. 判断主备无延迟方案，对于一些从库还没有收到的请求还是会有延迟
4. 配合 semi-sync，半同步只要一个从库返回ack就返回给客户端成功，但不能确保所有从库都同步完成
5. 等主库位点方案，主库更新完后执行show master status 得到当前主库执行到的File 和 Position，拿到信息后查询前去从库select master_pos_wait(File, Position, 1)；判断是否同步完成
6. GTID 方案，事务完成直接返回事务的 GTID，根据这个id去从库查询select wait_for_executed_gtid_set(gtid1, 1)；判断是否同步完成
#### 24. Join
让小表做驱动表、被驱动表有索引。  
如果被驱动表没有索引会走BNL算法，将驱动表加载到 join_buffer 中，将被驱动表中的数据一行行读出来与内存中的驱动表数据对比。  
如果被驱动表是个大表，会把冷数据的page加入到buffer pool（join_buffer 用了其中的内存），并且BNL要扫描多次，两次扫描的时间可能会超过1秒，使上节提到的分代LRU优化失效，把热点数据从buffer pool中淘汰掉，影响正常业务的查询效率。  
##### Join优化
Multi-Range Read 优化  
Batched Key Access：缓存读取多行传给被驱动表
##### BNL 算法的性能
除了给被驱动表加索引之外，还可以使用临时表，创建临时表然后加索引
#### 25. 临时表的应用
1. 临时表只能被创建它的 session 访问，对其他线程不可见。所以在这个 session 结束的时候，会自动删除临时表。
2. 临时表可以与普通表同名（还是不要这么做）。
3. session A 内有同名的临时表和普通表的时候，show create 语句，以及增删改查语句访问的是临时表。
4. show tables 命令不显示临时表。

##### 分表分库跨库查询
分库分表系统都有一个中间层 proxy，如果 sql 能够直接确定某个分表，这种情况是最理想的。  
但如果涉及到跨库，一般有两种方式：
1. 在 proxy 层的进程代码中实现排序，但对 proxy 的功能和性能要求较高。
2. 把各个分库拿到的数据，汇总到一个 MySQL 实例的一个表中，然后在这个汇总实例上做逻辑操作。如果每个分库的计算量都不饱和，那么直接可以在把临时表放到某个分库上。

#### 26. MySQL 什么时候会使用内部临时表？
1. 如果语句执行过程可以一边读数据，一边直接得到结果，是不需要额外内存的，否则就需要额外的内存，来保存中间结果；
2. join_buffer 是无序数组，sort_buffer 是有序数组，临时表是二维表结构；
3. 如果执行逻辑需要用到二维表特性，就会优先考虑使用临时表。比如，union 需要用到唯一索引约束， group by 还需要用到另外一个字段来存累积计数。

#### 27. group by使用的指导原则：
1. 如果对 group by 语句的结果没有排序要求，要在语句后面加 order by null；
2. 尽量让 group by 过程用上表的索引，确认方法是 explain 结果里没有 Using temporary 和 Using filesort；
3. 如果 group by 需要统计的数据量不大，尽量只使用内存临时表；也可以通过适当调大 tmp_table_size 参数，来避免用到磁盘临时表；
4. 如果数据量实在太大，使用 SQL_BIG_RESULT 这个提示，来告诉优化器直接使用排序算法得到 group by 的结果。

#### 28. insert 唯一键冲突
执行相同的 insert 语句，发现了唯一键冲突，加上读锁（Next-key lock）。session A 回滚，session B 和 session C 都试图继续执行插入操作，都要加上插入意向锁（LOCK_INSERT_INTENTION）。  
https://blog.csdn.net/varyall/article/details/80219459
#### 29. 怎么最快地复制一张表
1. mysqldump 方法
2. 导出 CSV 文件
3. mysql5.6 物理拷贝

假设我们现在的目标是在 db1 库下，复制一个跟表 t 相同的表 r：
1. 执行 create table r like t，创建一个相同表结构的空表；
2. 执行 alter table r discard tablespace，这时候 r.ibd 文件会被删除；
3. 执行 flush table t for export，这时候 db1 目录下会生成一个 t.cfg 文件；
4. 在 db1 目录下执行 cp t.cfg r.cfg; cp t.ibd r.ibd；这两个命令；
5. 执行 unlock tables，这时候 t.cfg 文件会被删除；
6. 执行 alter table r import tablespace，将这个 r.ibd 文件作为表 r 的新的表空间，由于这个文件的数据内容和 t.ibd 是相同的，所以表 r 中就有了和表 t 相同的数据。

#### 30. 分区表
innodb 只会锁一个分区，而 MyISAM 会锁所有的。
##### 应用
分区表的一个显而易见的优势是对业务透明，相对于用户分表来说，使用分区表的业务代码更简洁。还有，分区表可以很方便的清理历史数据。  
按照时间分区的分区表，就可以直接通过 alter tablet drop partition …这个语法删掉分区，从而删掉过期的历史数据。

#### 31. explain
##### Extra
1. Using index，表示这个语句使用了覆盖索引，选择了索引 a，不需要回表；
2. Using temporary，表示使用了临时表；
3. Using filesort，表示需要排序。