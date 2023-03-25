## **delete与is_deleted**

`DELETE`操作**并不会直接将数据从磁盘中删除**，而是会对这一行做删除标记，磁盘空间不会释放，但下次`INSERT`操作时，可以重用这部分空间。或者也可以在`DELETE`操作以后使用 `OPTIMIZE TABLE table_name` ，立刻释放磁盘空间。频繁的`DELETE`会导致索引频繁断裂，同时造成大量的存储碎片。在某些场景下，可以考虑使用逻辑删除，即设计一个删除的标志位is_deleted或do_delete等。

### 逻辑删除的唯一性索引问题

在逻辑删除的场景下，当我们有唯一性索引时，会遇到唯一性约束问题。具体来讲，当我们标记删除某条记录A后，再次插入与A记录唯一性字段相同的记录B时，会导致记录B无法插入。这个问题的解决办法，一般是将唯一性索引与删除标志组合起来形成新的唯一性索引。需要注意的是，假设某表的唯一性索引为fieldA_isDeleted，如果当前有两条fieldA相同的记录，其中一条已被标记为删除，另一条未删除，当我们想把后一条记录也标记删除的时候，不恰当的标记删除用法会导致此记录无法被标记删除。此时应该考虑将标记删除设置为null值，即标志位为null值代表记录被删除。参考下图。

> UNIQUE KEY `index_shop_user_doc_id` (`order_id`,`shop_user_id`,`doc_id`) USING BTREE

![img](https://notes-docker-images.oss-cn-beijing.aliyuncs.com/img/70b0afca2475d9b0a9fa5d2a6bcf1c1d.png)
