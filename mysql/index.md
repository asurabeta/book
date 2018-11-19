## Multi-Range Read (MRR)
`MRR 优化的目的是减少磁盘的随机访问， 将访问尽量转换为较为有序的数据访问，MRR 优化可用于 range， ref，eq_ref 类型的查询`

`MRR 优化有以下几个好处：`

- MRR 让数据库访问变得较为顺序。在查询辅助索引时，首先根据得到的查询结果，按照主键排序，并按照主键排序的顺序按照书签查找。
- 减少缓存池中页被替换的次数。
- 批量处理对键值的查询操作
  
 `对于InnoDB和MyISAM 存储引擎的范围查找和JOIN查询操作，MRR的工作方式如下`

 -  将查询得到的辅助索引放到一个缓存中，这是缓存中的数据是按照辅助索引键值排序的
 -  将缓存中的键值根据RowID进行排序
 -  根据RowID的排序顺序来访问实际数据文件


## Index Condition Pushdown (ICP)

`在支持ICP后，MySQL在取出索引的同时，判断是否可以进行WHERE条件的过滤，也就是将WHERE的部分过滤操作放在存储引擎层。ICP 优化支持MyISAM和InnoBD 存储引擎。当优化器选择ICP优化时，可在执行计划Extra看到Using index condition`

## InnoDB 存储引擎的hash算法

`InnoDB hash算法冲突机制采用链表方式，hash函数采用除数散列方法， 在缓存池中的Page页都有一个chain指针， 它指向相同hash函数值得页。 而对除法散列， m的取值要略大于2倍的缓存池页数量的质数。例如：当参数innodb_buffer_pool_size的大小为10M， 则共有640个16KB的页。对于缓存池内的hash表来说，需要分配 640*2=1280个槽，但由于1280不是质数, 需要比1280略大的质数，应该是1390，所以在启动时会分配1390个槽的hash表，用来查询缓冲池中的页。`

`hash页转换为自然数： InnoBD存储引擎表空间都有一个space_id， 用户要查询的应该是表空间的某个连续的16k的页，即偏移量offset， 关键字K=space_id<<20 + space_id + offset, 然后通过除法散列到各个槽中。`