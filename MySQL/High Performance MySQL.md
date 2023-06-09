## 高性能MySQL5.7笔记

### 第一章 MySQL架构与历史

​		本章概要地描述了MySQL的服务器架构、各种储存引擎之间的主要区别，以及这些区别之间的重要性。==MySQL最重要，最与众不同的特性是它的存储引擎架构，这种架构的设计将查询处理（Query Processing）及其他系统任务（Server Task）和数据的存储/提取相分离==。这种处理和存储分离的设计可以在使用时根据性能、特性，以及其他需求来选择数据存储的方式。

#### 1.1 MySQL逻辑架构

​		下图为MySQL服务器逻辑架构图。

- 最上层的服务并非MySQL独有的，大多数基于网络的C/S的工具或服务都有类似的架构。比如连接处理、授权认证、安全等。
- 第二层包含了大多数MySQL的核心服务功能，包括查询解析、分析、优化、缓存以及所有的内置函数（日期、字符串、数字、加密）等函数，所有的跨存储引擎的功能都在这一层实现：存储过程、触发器、视图等。
- 第三层包含了存储引擎。存储引擎负责MySQL中数据的存储和提取，每个存储引擎都有它的优势和劣势。服务器通过API与存储引擎进行通信，这些接口屏蔽了不同存储引擎之间的差异，使得这些差异对于上层查询过程透明。API中包含几十个底层函数，用于执行诸如“开始一个事务”或者“根据主键提取一行记录”等操作。不同存储引擎之间不会相互通信，而只是简单地响应上层服务器的请求。

![MySQL服务器逻辑架构图](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/MySQL_Architecture.svg)

##### 1.1.1 连接管理与安全性

​		每个客户端都会在服务器进程中拥有一个线程，这个连接的查询只会在这个单独的线程中执行，MySQL5.5或更新的版本支持线程池，可以使用少量的线程来服务大量的连接，减少CPU切换线程所消耗的服务器资源。

​		客户端连接至MySQL服务器时，服务器通过用户名、密码、主机信息的方式进行认证，如果使用了安全套接字（SSL）的方式连接，还可以使用X.509证书认证。客户端连接之后，服务器进一步验证客户端的权限（如是否拥有某个表的读、写权限）。

##### 1.1.2 优化与执行

​		MySQL会对查询进行解析，并创建内部数据结构（解析树），对其进行各种优化。

​		对于SELECT语句，在解析查询之前，服务器会先检查查询缓存（Query Cache），如果能找到对应的查询，则直接命中。

#### 1.2 并发控制

##### 1.2.1 读写锁

概念：

- 读锁是共享的，或者说是相互步阻塞的。多个用户同时读一个资源，互不干扰。
- 写锁是排他的，换句话说写锁会阻塞其他的读锁和写锁，这样才能保证同一时间只有一个用户执行写入，同时保证正在被写入的资源不被以不确定的方式被读取。

写锁的优先级高于读锁，因此一个写锁请求可能会被插入到读锁队列的前面，实际上，MySQL中每时每刻都在发生锁定，大多数时候，锁的内部管理是透明的。

##### 1.2.2 锁粒度

​		通过让锁定对象更有选择性，提高共享资源的并发性。任何时候，在给定的资源上，锁定的数据量越少，则系统的并发程度就越高，只要相互不发生冲突。但锁本身也需要消耗资源，获得锁、检查锁、释放锁都会增加系统的开销。

​		锁策略，即在锁的开销和数据的安全性之间需求平衡。一般的商业数据库系统未提供更多的选择，一般都是在表上加行级锁（row-level lock），而MySQL提供了更多的选择。每种MySQL存储引擎都可以实现自己的锁策略和锁粒度。在存储引擎的设计中，锁策略是非常重要的决定。下面介绍两种最重要的锁策略：

- **表锁（table lock）**

  表锁是最基本的锁策略，并且是开销最小的策略。表锁将会锁定整张表。一个用户对表进行写操作（插入、删除、更新等）前，需要先获得写锁，这会阻塞其他用户对表的所有读写操作。只有没有写锁的时候，其他读取的用户才能获得读锁，读锁之间互不阻塞。特定场景下，表锁也可能有良好的性能。

> 尽管存储引擎可以管理自己的锁，MySQL本身还是会使用各种有效的表锁来实现不同的目的。例如，服务器会为诸如ALTER TABLE之类的语句使用表锁，而忽略存储引擎的锁策略。

- **行级锁（row lock）**

  行级锁可以最大程度地支持并发处理（同时带来了最大的锁开销）。众所周知，在InnoDB和XtraDB，以及其他一些存储引擎实现了行级锁。行级锁只在存储引擎层实现，而MySQL服务器层没有实现。

#### 1.3 事务

​		事务就是一组原子性的SQL查询，或者说一个独立的工作单元。如果数据库可以应用该组的全部语句，那么就执行该组查询。如果其中有任意一条语句因为崩溃或其他原因无法执行，那么所有的语句都不会执行。换句话说，事务中的语句，要么全部执行，要么全部都不执行。

​		以经典的银行应用来解释事务的重要性。假设银行的数据库有两张表：支票（checking）表和储蓄（savings）表。现在要从用户Jane的支票账户转200元到储蓄账户，那么至少需要三个步骤：

1. 检查支票账户中余额是否高于200元。
2. 从支票账户余额中减去200元。
3. 在储蓄账户余额中添加200元。

上诉三个步骤必须打包在一个事务中，任何一个步骤失败，都必须回滚所有的步骤。

可以使用START TRANSACTION语句开始一个事务，然后要么使用COMMIT提交事务将修改的数据持久保留，要么使用ROLLBACK撤销所有的修改。

```mysql
START TRANSACTION;
SELECT balance FROM checking WHERE customer_id = 123456;
UPDATE checking SET balance = balance - 200.00 WHERE customer_id = 123456;
UPDATE savings SET balance = balance + 200.00 WHERE customer_id = 123456;
COMMIT;
```

一个运行良好的事务处理系统，必须通过严格的ACID测试。ACID表示原子性（Atomicity）、一致性（consistency）、隔离性（isolation）和持久性（durability）。

**原子性（Atomicity）**

​		一个事务必须被视为一个不可分割的最小工作单元，整个事务中的操作要么全部执行，要么全部失败回滚。

**一致性（consistency）**

​		数据库总是从一个一致性状态转换到另一个一致性状态。在前面的例子中，即使在低3条或第4条语句之间系统崩溃了，账户中也不会白白损失200元，因为事务最终未能提交，所以事务中的修改也不会保存到数据库中。

**隔离性（isolation）**

​		通常来说，一个事务的修改在未提交前，对其他事务是不可见的。在前面的例子中，当第3条语句执行后，即使另一个事务读取支票账户的余额，其看到的余额并没有减去200元。后面讨论隔离级别（Isolation level）的时候，会进一步讨论为什么是“通常来说”。

**持久性（durability）**

​		一旦事务提交，则其所做的修改都会永久保存到数据库中。即使此时系统崩溃，修改也不会丢失。持久性比较模糊，后面将进一步讨论。

> 事务处理也会增加系统的开销。对于一些不需要事务的查询类应用，选择一个非事务型的存储引擎，可以获得更高的性能。

##### 1.3.1 隔离级别

​		SQL标准中定义了四种隔离级别，每一种级别都规定了事务中的所做的修改，哪些是事务内和事务间可见的，哪些是不可见的。较低级别的隔离通常可以执行更高的并发，系统开销也更低。

- **READ UNCOMMITTED（未提交读）**

  在**READ UNCOMMITTED**级别，事务中的修改，即使没有提交，对其他事务也是可见的。事务可以读取为提交的数据，这也被称为脏读（Dirty Read）。这个级别会导致许多问题，从性能上来说，它也不必其他级别好太多，但却缺乏其他级别的很多好处，除非真的有必要，在实际中很少用。

- **READ COMMITED（提交读）**

  大多数数据库系统默认隔离级别都是**READ COMMITED**（但MySQL不是）。**READ COMMITED**满足前面提到的隔离性的简单定义：一个事务开始时，只能“看见”已经提交的事务的修改。换句话说，一个事务从开始知道提交之前，所做的任何修改都其他事务都是不可见的。这个级别也可以称之为不可重复读（unrepeatable read），因为两次执行同样的查询，可能得到不一样的结果（指在一次事务中，由于其他事务的提交导致前一次查询和后一次查询得到的结果不一样）。

- **REPEATABLE READ（可重复读）**

  **REPEATABLE READ**解决了不可重复读的问题。该级别保证了在同一事务中多次读取同样记录的结果是一致的。但是理论上，这个级别无法解决幻读的问题。所谓幻读，指的是当某个事务在读取某个范围内的记录时，另一个事务插入了新的纪录，当之前的事务再次读取该范围内的纪录时，会产生幻行。InnoDB和XtraDB存储引擎通过多版本并发控制（MVVC）解决了幻读的问题。==MySQL的默认事务隔离级别是可重复读==

- **SERIALIZABLE（可串行化）**

  **SERIALIZABLE**是最高的隔离级别。通过强制事务串行执行，避免了前面说的幻读的问题。**SERIALIZABLE**会在读取的每一行数据上都加上锁，所以可能导致大量的超时和锁争用的问题。实际上很少使用这个级别，只有非常需要确保数据一致性且可以接受并发的情况下使用。

##### 1.3.2 死锁

​		**死锁指两个或者多个事务在同一资源上相互占有，并请求锁定对方占用的资源，从而导致恶性循环的现象。**当多个事务试图以不同顺序锁定资源时，或多个事务同时锁定同一个资源时，都会产生死锁。

例如：

```mysql
-- 事务一
START TRANSACTION;
UPDATE StockPrice SET close = 45.50 WHERE stock_id = 4 AND date = '2002-05-01';
UPDATE StockPrice SET close = 19.80 WHERE stock_id = 3 AND date = '2002-05-02';
COMMIT;

-- 事务二
START TRANSACTION;
UPDATE StockPrice SET high = 20.10 WHERE stock_id = 3 AND date = '2002-05-02';
UPDATE StockPrice SET high = 10.80 WHERE stock_id = 4 AND date = '2002-05-01';
COMMIT;
```

如果两个事务都执行了第一条UPDATE，锁定了该行的数据，接着尝试执行第二条UPDATE语句，由于资源被对方锁定，因此两个事务相互等待对方释放锁，陷入死循环。

为了解决这个问题，数据库系统实现了各种死锁检测和死锁超时机制。越复杂的系统，比如InnoDB存储引擎，越能检测到死锁的循环依赖，并立即返回一个错误；或者当查询的时间达到锁等待超时的设定后放弃锁请求（这种方式不太友好）。InnoDB目前的处理死锁的方法是，将持有最少行级排他锁的事务进行回滚（这是相对比较简单的死锁回滚算法）。

> 锁的行为和顺序是和存储引擎相关的。以同样的顺序执行语句，有些存储引擎会产生死锁，有些则不会。
>
> 死锁的产生有双重原因：有些是真正的数据冲突，有些是存储引擎的实现方式。

##### 1.3.3 事务日志

使用事务日志，存储引擎在修改表的数据时只需要修改其内存拷贝，再将该修改行为记录到持久在硬盘上的事务日志中，而不用每次都将修改的数据本身持久到磁盘。事务日志采取追加的方式，因此写日志的操作是磁盘上一小块区域内的顺序I/O，而非随机I/O，所以比较快。事务日志持久后，内存中的数据在后台可以慢慢地刷回到磁盘，目前大多数存储引擎都是这样实现的，称之为预写式日志，修改数据需要写两次磁盘。

> 如果数据的修改记录到日志并持久化，但数据未写会磁盘，此时系统崩溃，存储引擎在重启时能够自动恢复这部分修改的数据。具体的方法视存储引擎而定。

##### 1.3.4 MySQL中的事务

MySQL提供两种事务型存储引擎：InnoDB和NDB Cluster。还有一些第三方的存储引擎：XtraDB和PBXT。

**自动提交**

MySQL默认自动提交。如果需要自定义事务，可以关闭自动提交。注意：在事务中，如果使用会导致大量数据改变的数据定义语句DDL，比如ALTER TABLE， TRUNCATE就会强制执行COMMIT提交当前的事务。除此之外，LOCK TABLES等其他语句也会导致相同的结果。

> MySQL可以通过执行**SET TRANSACTION ISOLATION LEVEL**来设置隔离级别，可以在配置中设置，也可以在回话中设置。

```mysql
mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

**事务中使用混合存储引擎**

MySQL服务层不管理事务，事务由更下层的存储引擎实现，所以在同一个事务中，使用多种存储引擎是不可靠的。

在事务中使用事务型（InnoDB）和非事务型（MyISAM）表，==正常提交==的情况下不会有什么问题。但如果需要回滚，非事务型的表上的变更无法回滚，这会导致数据库处于不一致的状态！

在非事务表上执行事务相关操作时，MySQL通常不报错也不提醒，有时在回滚的时候发出警告。

**隐式和显式锁定**

InnoDB采用两阶段锁定协议。在事务执行过程中，随时都可以执行锁定，锁只有在执行**COMMIT**或者**ROLLBACK**的时候才会释放，并且所有的锁是在同一时刻释放（隐式锁）。==MySQL也支持LOCK TABLES和UNLOCK TABLES语句，这是在服务器层面实现的，和引擎无关==。它们有自己的用途，但并不能替代事务处理。

> LOCK TABLES和事务之间相互影响的话，情况会比较复杂。因此，建议除了事务中禁用了AUTOCOMMIT可以使用LOCK TABLE外，其他时候不要显式地执行LOCK TABLES。

#### 1.4 多版本并发控制

MySQL的大多数事务型存储引擎实现的都不是简单的行级锁。基于提升并发性能的考虑，他们一般都同时实现了多版本并发控制（MVCC）。MVCC的实现，是通过保存数据在某个时间点的快照来实现的。

> 不同存储引擎MVCC实现方式不一样，典型的有乐观并发控制和悲观并发控制。

通过InnoDB的简化版行为来说明MVCC：

InnoDB的MVCC，是通过在每行记录后面保存两个隐藏的列来实现的，一个保存记录的创建版本号，一个保存记录的删除版本号。每开始一个事务，系统的版本号都会自动递增。事务开始时刻的系统版本号回作为事务的版本号，用于和查询到的记录的版本号进行比较。以可重复读为例：

- **SELECT**

  - InnoDB只查找版本号小于或等于当前事务版本的记录，这样可以确保事务读取到的行，要么在事务开始之前存在，要么在事务本身插入或修改的
  - 行的删除版本要么未定义，要么大于当前事务版本号。这可以确保事务读取到的行在事务开始之前未被删除

- **INSERT**

  InnoDB为新插入的每一行保存当前系统版本号作为行版本号

- **DELETE**

  InnoDB为删除的每一行保存当前系统版本号作为行删除标识

- **UPDATE**

  InnoDB插入一行新纪录，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来的行作为行删除标识

> MVCC只在可重复读和提交读两个隔离级别工作，其他两个级别一个是串行的，另一个总是最新的数据

#### 1.5 MySQL的存储引擎

文件系统中，MySQL将每个数据库（schema）保存为数据目录下的一个子目录。创建表时，MySQL会在数据库子目录下创建与表同名的frm文件用于保存表的定义，正因MySQL用文件系统的目录和文件来保存数据库和表的定义，因此是否区分大小写取决于系统是否区分。

> 表的定义是在MySQL服务层统一处理的。
>
> 注：对于InnoDB，表的Rows是估计值，对于MyISAM的其他一些，该值为精确值。

##### 1.5.1 InnoDB存储引擎

InnoDB被设计用来处理大量的短期（short-lived）事务，短期事务大部分情况是正常提交的，很少会被回滚。InnoDB采用MVCC来支持高并发，其默认隔离级别为可重复读，并通过间隙锁（next-key locking）策略防止幻读的出现。

##### 1.5.2 MyISAM存储引擎

**存储**

**MyISAM**会将表存储在两个文件中：数据文件和索引文件，分别以.MYD和.MYI为扩展名。MyISAM可以包含动态或者静态（长度固定）行。MySQL会根据表的定义来决定采用何种行格式。MyISAM表可以存储的行记录数，一般受限于可用的磁盘空间，或者操作系统中单个文件的最大尺寸。`不具有事务特性，适合于主要是SELECT和INSERT操作的应用，一般适合用作日志型的应用`

**特性**

- 加锁与并发

  MyISAM对整张表加锁，而不是针对行。读取时对需要读到的所有表加共享锁，写入时则对表加排他锁。但是在表有共享锁时，也可以往表中插入新纪录。

- 修复

  对于MyISAM表，MySQL可以手工或者自动执行检查和修复操作，这里的修复和事务恢复以及崩溃恢复是不同的概念。

- 索引特性

  对于MyISAM表，即使是BLOB和TEXT等长字段，也可以基于前500个字符创建索引。

- 延迟更新索引键（Delayed Key Write）

  创建MyISAM时，可以通过指定DELAY_KEY_WRITE的选项，使得每次修改时，不会立即将修改的索引数据写入磁盘，而是会写到内存中的键缓冲区，只有在清理键缓冲区或者关闭表的时候才会将对应的索引快写入到磁盘。这种方式极大提升性能，但是数据库崩溃会造成索引损坏，需要修复。

**压缩表**

如果表在创建并导入数据以后，不会再进行修改操作，那么这样的表或许适合采用MyISAM压缩表。压缩表是不能修改的（除非先解压缩，修改之后再次压缩）。压缩表可以极大地减少磁盘空间占用，因此也可以减少磁盘I/O，从而提升查询性能。

**性能**

MyISAM引擎设计简单，数据以紧密格式存储，所以在某些场景下的性能很好。但MyISAM也存在一些问题，其中最经典的还是表锁的问题。

##### 1.5.3 内建的其他存储引擎

- Archive引擎

  只支持**INSERT**和**SELECT**操作，在5.1之前也不支持索引。Archive引擎会缓存所有的写并利用*zlib*对插入的行进行压缩，所以比MyISAM表的磁盘I/O更少。但是每次SELECT查询都需要执行全表扫描。适合日志和数据采集类的应用，或者一些需要更快速的INSERT操作的场合。Archive引擎支持行级锁和专用的缓冲区，所以可以实现高并发的插入。

- Blackhole引擎

  没有实现任何的存储机制，它会丢弃所有插入的数据，不做任何保存，但是服务器会记录Blackhole表的日志。这种应用方式曾遇到很多问题，因此并不推荐。

- CSV引擎

  可以将普通的CSV文件作为MySQL的表来处理，但这种表不支持索引。此引擎支持在数据库运行时拷入或拷出文件，因此CSV引擎可以作为一种数据交换的机制，非常有用。

- Federated引擎

  访问其他MySQL服务器的代理，它会创建一个到远程MySQL服务器的客户端连接，并将查询传输到远程服务器执行，然后提取或发送需要的数据。由于经常带来问题，所以此引擎默认是禁用的。

- Memory引擎

  如果需要快速访问数据，且这些数据不会被修改，重启之后丢失也没关系，那么使用Memory表是非常有用的。Memory表至少比MyISAM表快一个数量级，因为所有的数据都保存在内存中，不需要进行磁盘I/O。Memory表的结构在重启以后还会保留，但数据会丢失。

  Memory表在很多场景可以发挥好的作用：

  - 用于查找或者映射表，例如邮编和省市映射的表
  - 用于缓存周期性聚合数据的结果
  - 用于保存数据分析中产生的中间数据

  Memory表支持Hash索引，因此查找操作非常快。==Memory表是表级锁，因此并发写入的性能较低。==它不支持BLOB和TEXT类型的列，每行的长度固定，所以即使指定了varchar也会转换为char。

- Merge引擎

  该引擎已被放弃

- NDB集群引擎

  详见后续章节的内容

##### 1.5.4 第三方存储引擎

- OLTP类引擎
  - XtraDB
  - PBXT
  - TokuDB
  - RethinkDB
- 面向列的存储引擎
  - Infobright
  - InfiniDB

##### 1.5.5 选择合适的存储引擎

> 除非需要用到某些InnoDB不具备的特性，并且没有其他办法可以替代，否则都应该优先选择InnoDB引擎。

选择存储引擎优先考虑的因素：

- ___事务___。判断应用是否需要事务支持，InnoDB(XtraDB)是目前最稳定且经过验证的选择。
- ___备份___。如果可以定期关闭服务器来执行备份，那么就不需要考虑此因素。否则，应该选择InnoDB用于热备份。
- ___崩溃恢复___。考虑到崩溃后发生损坏的概率MyISAM大于InnoDB，因此，选择InnoDB会是个不错的选择。
- ___特有的特性___。例如：MyISAM支持地理空间搜索。





## MySQL日常使用

### 字符串的存储

 **1、关于char，varchar，text**

​	1、char（n）和varchar（n）中括号中n代表字符的个数，并不代表字节个数，所以当使用了中文的时候(UTF8)意味着可以插入m个中文，但是实际会占用m*3个字节。

​	2、同时char和varchar最大的区别就在于char不管实际value都会占用n个字符的空间，而varchar只会占用实际字符应该占用的空间+1，并且实际空间+1<=n。

​	3、超过char和varchar的n设置后，字符串会被截断。

​	4、char的上限为255字节，varchar的上限65535字节，text的上限为65535。

​	5、char在存储的时候会截断尾部的空格，varchar和text不会。

​	6、varchar会使用1-3个字节来存储长度，text不会。



空间方面：

​	从官方文档中我们可以得知当varchar大于某些数值的时候，其会自动转换为text，大概规则如下：

​	— 大于varchar（255）变为 tinytext

​	— 大于varchar（500）变为 text

​	— 大于varchar（20000）变为 mediumtext

​	所以对于过大的内容使用varchar和text没有太多区别。



性能方面：

​	索引会是影响性能的最关键因素，而对于text来说，只能添加前缀索引，并且前缀索引最大只能达到1000字节。 而貌似varhcar可以添加全部索引，但是经过测试，其实也不是。由于会进行内部的转换，所以long varchar其实也只能添加1000字节的索引，如果超长了会自动截断。



所以我们认为当超过**255**的长度之后，使用**varchar**和**text**没有本质区别，只需要考虑一下两个类型的特性即可。（主要考虑**text**没有默认值的问题）



### ON DUPLICATE KEY UPDATE

​		使用ON DUPLICATE KEY UPDATE，最终如果插入了一个新行，则受影响的行数是1，如果修改了已存在的一行数据，则受影响的行数是2。



### CAST函数

使用cast将字符串转数字时，若字符串为数字开头，如`123abcd`，`12abc123`等时，函数将会返回开头的数字，规则如下：

`123abcd`	=>	`123`，`12abc123`	=>	`12`。如果开头不为数字，如"abc12"，"abc"等，将返回数字0。

使用场景：

数据中存在名为name的字段，此字段有"1小"，"1大"，"2小"，"2大"等数据，直接使用order by对此字段进行排序，将会得到10大，10小，1大，1小，2大，2小等的顺序，为了得到符合业务需要的排序，对此字段进行cast()运算再进行排序，得到预期的排序结果，同时，再通过对cast()结果排序后，再对原数据进行排序来达到"小"和"大"之间的排序

拓展：

​	题目：如果要把字段中的数字分离出来，例如：将"12abcd35"中的数字提取出来，得到1235，如何做到以上效果呢？

​	答：可以将整个字符串分离，利用row_number()和substr()函数的运用，将所有字符分离，再通过ascii()区分出数字，再将数字通过原有顺序进行排序，最后将得到的结果通过cast()处理得到最终结果。



### 时间函数

mysql中三个获取时间的函数，分别是`now()` `current_timestamp()` `sysdate()`。其中`now()` `current_timestamp()` 是同义的，都返回语句运行的时间。而`sysdate()`返回此函数运行的时间。举个例子：

```sql
SELECT
NOW(), CURRENT_TIMESTAMP(), SYSDATE(),
SLEEP(1),
NOW(), CURRENT_TIMESTAMP(), SYSDATE()
```

![image-20220311165715148](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/image-20220311165715148.png)

### docker起mysql服务相关问题

docker起的mysql默认开启了SSL证书认证连接，默认字符集是Latin1，因此创建的数据库默认情况下也是这个字符集，数据库的字符集可以通过以下命令来修改：

```mysql
ALTER DATABASE basename character SET utf8mb4;
```

而mysql默认字符集和SSL连接禁用需要去配置文件中修改，在my.cnf中添加

```shell
[mysqld]
skip_ssl
character-set-server=utf8mb4
```

### 关于mysql的索引

MySQL有FULLTEXT、NORMAL、SPATIAL、UNIQUE四种索引（PRIMARY实际上属于key）。以下是他们的特点和性质：

#### NORMAL索引

普通索引，索引列没有任何限制。

#### UNIQUE索引

唯一索引，索引值必须唯一，但允许为NULL

> primary key = unique(唯一) + not null(不为空)

#### Full Text索引

表示全文索引，在检索长文本的时候，效果最好，短文本建议使用Index,但是在检索的时候数据量比较大的时候，现将数据放入一个没有全局索引的表中，然后在用Create Index创建的Full Text索引，要比先为一张表建立Full Text然后在写入数据要快的很多

#### SPATIAL索引

空间索引是对空间数据类型的字段建立的索引，MYSQL中的空间数据类型有4种，分别是GEOMETRY、POINT、LINESTRING、POLYGON。MYSQL使用SPATIAL关键字进行扩展，使得能够用于创建正规索引类型的语法创建空间索引。创建空间索引的列，必须将其声明为NOT NULL，空间索引只能在存储引擎为MYISAM的表中创建

#### 在实际操作过程中，应该选取表中哪些字段作为索引？

为了使索引的使用效率更高，在创建索引时，必须考虑在哪些字段上创建索引和创建什么类型的索引，有：

1. 选择唯一性索引
2. 为经常需要排序、分组和联合操作的字段建立索引
3. 为常作为查询条件的字段建立索引
4. 限制索引的数目
5. 尽量使用数据量少的索引
6. 尽量使用前缀来索引
7. 删除不再使用或者很少使用的索引
8. 经常更新修改的字段不要建立索引（针对mysql说，因为字段更改同时索引就要重新建立，排序，而Orcale好像是有这样的机制字段值更改了，它不立刻建立索引，排序索引，而是根据更改个数，时间段去做平衡索引这件事的）
9. 不推荐在同一列建多个索引

#### B-Tree和Hash的区别

从名字可以看出，B-Tree是B+树的数据结构，Hash是哈希表的数据结构，根据这两种数据结构的特点，我们可以知道：

- Hash适合于判断等于与否，不能用于排序，不能用于范围查询
- Hash索引中的复合索引为整个索引键的哈希值，因此不能用于部分索引查询
>  最左匹配原则

![explain](https://raw.githubusercontent.com/Hxhao2000/ImagesBed/master/Images/explain_xmind.png)

### FIND_IN_STR和LOCATE函数
