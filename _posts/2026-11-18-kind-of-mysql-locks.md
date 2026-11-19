---
layout: post
title:  面试官：MySQL中有哪几种锁，各自的特点是什么？MyISAM和InnoDB在锁机制的区别？行级锁实现原理？死锁的产生与排查？
date:   2026-11-18 10:43:08
categories: 数据库
tags: 间隙锁
---

* content
{:toc}

面试题概览：

- MySQL中有哪几种锁？它们各自的特点是什么？
- 说说看MyISAM和InnoDB在锁机制上的主要区别？
- MySQL中InnoDB存储引擎的行级锁实现原理？
- MySQL中记录锁、间隙锁和临键锁的区别和使用场景？
- MySQL中对于不同的隔离级别，行锁采用的加锁方式有什么不同?
- MySQL中的死锁是如何产生的，请给出示例？如何避免和解决死锁问题？

面试官：MySQL中有哪几种锁？它们各自的特点是什么？

![](https://mmbiz.qpic.cn/mmbiz_jpg/B220micXXwONiaOyEa0qAsw2WTibosEvv4wRxBsCyFPjM97sAKlzUaP0DLsCt5TpfN7HWQwKaBVNu5Fjl7RSpsQFQ/640?wx_fmt=jpeg&from=appmsg)

MySQL中的锁机制是数据库并发控制的重要组成部分，主要包括以下几种锁，以及它们的特点：

### 一、全局锁

- **概念**：全局锁是对整个数据库实例加锁，加锁后整个实例就处于只读状态，后续的MDL（元数据锁）、DDL（数据定义语言）语句、更新操作的事务提交语句都将被阻塞。
- **特点**：
- 锁范围大，影响整个数据库实例。

  常用于全库的逻辑备份、全库的导出等场景，以获取一致性视图，保证数据的完整性。

  加锁后，数据库实例变为只读状态，无法执行更新操作。

### 二、表级锁

- **概念**：表级锁是对当前操作的整张表加锁。
- **特点**：
- 锁定粒度较大，但开销较小，加锁速度快。

  适用于以查询为主，只有少量按索引条件更新数据的应用，如Web应用。

  容易出现锁冲突，并发度较低。

  MyISAM和InnoDB存储引擎都支持表级锁定。

### 三、页级锁

- **概念**：页级锁是锁住某一页的数据（通常是16KB左右）。
- **特点**：
- 锁定粒度介于表级锁和行级锁之间。

  开销和加锁时间也介于表级锁和行级锁之间。

  会出现死锁现象。

  并发度一般，适用于一些中等并发度的应用场景。

### 四、行级锁

- **概念**：行级锁是锁住某一行的数据。
- **特点**：
- 锁定粒度最小，锁冲突的概率最低，并发度最高。

  但加锁慢、开销大，且容易出现死锁现象。

  适用于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用，如在线事务处理（OLTP）系统。

  InnoDB存储引擎支持行级锁定。

### 五、按模式分类的锁

- **乐观锁**

  乐观锁并不是数据库自带的锁，而是应用层面的锁。它假定大概率不会发生并发更新冲突，在访问、处理数据的过程中不加锁，只在更新数据时根据版本号或时间戳判断是否有冲突。如果冲突则处理，无冲突则提交事务。乐观锁适用于系统并发量非常大的场景。
- **悲观锁**

  悲观锁假定大概率会发生并发更新冲突，因此在访问、处理数据前就加排他锁，在整个数据处理过程中锁定数据，事务提交或回滚后才释放锁。悲观锁为数据处理的安全提供了保证，但会降低并发性能。

### 六、按属性分类的锁

- **共享锁（S锁、读锁）**

  允许读，但不能写。共享锁可以与共享锁一起使用。
- **排他锁（X锁、写锁）**

  不能允许读，也不能允许写。排他锁不能与其他锁一起使用。在MySQL中，update、delete、insert、alter等写的操作默认都会加上排他锁。

### 七、InnoDB存储引擎特有的锁

- **意向锁**：InnoDB为了支持多粒度的锁，即允许行级锁和表级锁共存，而引入意向锁。意向锁是指未来的某个时刻，事务可能要加共享/排他锁，先提前声明一个意向。

- **记录锁（Record Lock）**：锁住的是单条索引记录，防止其他事务读取或修改被锁住的记录。
- **间隙锁（Gap Lock）**：锁定索引记录之间的间隙，即范围查询的记录。间隙锁的主要目的是为了防止其他事务在间隔中插入数据，以导致“不可重复读”。
- **临键锁（Next-Key Lock）**：是记录锁和间隙锁的组合，锁的范围既包含记录又包含索引区间。默认情况下，InnoDB使用临键锁来锁定记录。临键锁的主要目的也是为了避免幻读（Phantom Read）。
- **自增锁（Auto-Inc Lock）**：专门用于管理自增列，防止自增值竞争导致的冲突。在插入新记录时，锁住整个表，使得自增值不受混乱影响。

###

面试官：说说看MyISAM和InnoDB在锁机制上的主要区别？

MyISAM和InnoDB是MySQL数据库中两种常用的存储引擎，它们在锁机制上存在显著的区别。

### 一、锁类型与支持范围

**MyISAM**

1. MyISAM只支持表级锁（Table-Level Locking）。

   表级锁意味着在对表进行读或写操作时，会锁定整个表，其他事务无法同时对同一表进行读或写操作。
2. **InnoDB**
3. InnoDB支持行级锁（Row-Level Locking）和表级锁。

   行级锁允许事务仅锁定需要修改的数据行，而不是整个表，从而提高了并发性能。

   InnoDB还支持其他类型的锁，如间隙锁（Gap Lock）和临键锁（Next-Key Lock），这些锁用于防止幻读和确保数据一致性。

### 二、锁的特点与性能

**MyISAM的表级锁**

1. **开销小、加锁快**

   由于锁定的是整个表，因此加锁的开销相对较小，加锁速度也较快。
2. **不会出现死锁**

   由于MyISAM的表级锁是互斥的，一个事务在持有表级锁时，其他事务无法获取该表的锁，因此不会出现死锁。
3. **锁粒度大、并发度低**

   由于锁定的是整个表，锁粒度较大，当多个事务需要同时访问同一表时，容易发生锁冲突，导致并发性能较低。
5. **InnoDB的行级锁**
7. **开销大、加锁慢**

   由于锁定的是数据行，需要维护更多的锁信息，因此加锁的开销相对较大，加锁速度也较慢。
8. **可能出现死锁**

   当多个事务以不同的顺序访问相互关联的数据行时，可能会出现死锁。InnoDB通过死锁检测和解决机制来处理这种情况。
9. **锁粒度小、并发度高**

   由于锁定的是数据行，锁粒度较小，当多个事务需要同时访问同一表的不同行时，可以并行处理，提高了并发性能。

### 三、锁的应用场景

1. **MyISAM的表级锁**

- 适用于读操作较多、写操作较少的场景。因为读操作不会阻塞其他读操作（但会阻塞写操作），所以在读操作频繁的情况下，MyISAM的表级锁可以提供较好的性能。
- 适用于不需要事务支持或并发性能要求不高的场景。

2. **InnoDB的行级锁**

- 适用于读写操作频繁、并发性能要求高的场景。因为行级锁可以允许多个事务并行处理不同的数据行，从而提高了并发性能。
- 适用于需要事务支持、数据一致性要求高的场景。InnoDB的行级锁和事务机制可以确保数据的一致性和完整性。

面试官：MySQL中InnoDB存储引擎的行级锁实现原理？

### 一、行级锁的实现方式

InnoDB的行级锁是通过给索引的索引项加锁来实现的。当事务对某行数据进行操作时，InnoDB会为该行数据对应的索引项加锁，以确保其他事务无法同时修改该行数据。

### 二、行级锁的工作机制

1. **加锁过程**：

- 当事务执行`UPDATE`、`DELETE`等修改数据的操作时，InnoDB会自动为涉及的数据行加排他锁（X锁）。
- 当事务执行`SELECT ... FOR UPDATE`或`SELECT ... LOCK IN SHARE MODE`等查询操作时，InnoDB会根据查询条件为相应的数据行加共享锁（S锁）或排他锁（X锁）。

2. **锁释放**：

- 事务提交（`COMMIT`）后，InnoDB会自动释放该事务持有的所有锁。
- 事务回滚（`ROLLBACK`）后，InnoDB也会释放该事务持有的所有锁。

3. **锁升级**：

- 在InnoDB中，锁通常不会直接进行升级或降级操作。但是，如果事务在持有共享锁的情况下尝试对数据进行修改，InnoDB会自动将共享锁升级为排他锁。

面试官：简述MySQL中记录锁、间隙锁和临键锁的区别和使用场景。

MySQL中的Record Lock（记录锁）、Gap Lock（间隙锁）和Next-Key Lock（临键锁）是InnoDB存储引擎提供的不同锁机制，以下是这三种锁的区别和使用场景的详细简述：

### 一、Record Lock（记录锁）

1. **定义**：Record Lock是MySQL InnoDB存储引擎提供的行级锁机制的一部分，直接作用于存储引擎层面的索引记录上，即锁定单个或多个行记录。

   ![](https://mmbiz.qpic.cn/mmbiz_png/B220micXXwONiaOyEa0qAsw2WTibosEvv4wVLvsWVknjXBKOeKMCZvtWYrH4PZvT2YkcGQOxcra06ydreZQBegpmg/640?wx_fmt=png&from=appmsg)
2. **作用**：控制对单个数据行的并发访问，减少数据操作冲突，提高系统的并发能力。
3. **使用场景**：当一个事务需要对某行数据进行修改或读取时，会对该行的索引记录加锁。
4. **特点**：
5. 锁定的是索引记录，而非记录本身。

   如果表没有定义索引，InnoDB会隐式创建一个聚簇索引（也称为主键索引），并基于该索引加锁。

### 二、Gap Lock（间隙锁）

1. **定义**：Gap Lock是InnoDB存储引擎在可重复读隔离级别下为了防止幻读而引入的锁机制，用于锁定索引记录之间的间隙。

   ![](https://mmbiz.qpic.cn/mmbiz_png/B220micXXwONiaOyEa0qAsw2WTibosEvv4wFhlbcp5Os5WO2xibn4xRzCM4NXAkHKEzT0Ga5s7dvicHYzj658EAmNYQ/640?wx_fmt=png&from=appmsg)
2. **作用**：确保索引记录的间隙不变，防止其他事务在间隙中插入新的记录，从而避免幻读现象。
3. **使用场景**：当执行范围查询并希望防止其他事务在查询范围内插入新记录时，Innodb会使用Gap Lock。
4. **特点**：
5. 锁定的是索引记录之间的间隙，而不是具体的数据行。

   Gap Lock是共享的，多个事务可以在同一间隙上持有Gap Lock，但如果有事务在某个间隙上持有Gap Lock，其他事务就不能在这个间隙中插入新的记录。

   主要用于解决可重复读模式下的幻读问题。

### 三、Next-Key Lock（临键锁）

1. **定义**：Next-Key Lock是Record Lock和Gap Lock的组合，既锁定索引记录本身，又锁定索引记录之间的间隙。

   ![](https://mmbiz.qpic.cn/mmbiz_png/B220micXXwONiaOyEa0qAsw2WTibosEvv4w4LVG6RcnFzw7XS099HPzJ6eE341LsmTnOg2WxmypdJLeBkOXickautA/640?wx_fmt=png&from=appmsg)
2. **作用**：同时防止其他事务对同一行数据进行修改和在该行数据的前后间隙中插入新的数据行，从而有效地避免幻读现象的发生。
3. **使用场景**：在可重复读隔离级别下，InnoDB默认使用Next-Key Lock来防止幻读。当执行范围查询或更新时，会自动为这个范围内的每个数据行加上行锁，同时在数据行之间的间隙上加上Gap Lock。
4. **特点**：

- 锁定的范围是当前查询的行及其“前后”的间隙。
- 可以有效防止其他事务在当前查询的范围内插入新数据或修改已有数据。

以下是使用了记录锁、间隙锁和临键锁的SQL示例：

1. 记录锁（Record Lock）示例

```
-- 假设有一个名为user的表，包含id和name字段CREATE TABLE `user` (    `id` INT AUTO_INCREMENT PRIMARY KEY,    `name` VARCHAR(50) NOT NULL);-- 插入一些示例数据INSERT INTO `user` (`name`) VALUES ('Alice'), ('Bob'), ('Charlie');
-- 开启事务1START TRANSACTION;
-- 事务1尝试更新id为2的记录，此时会对该记录加记录锁UPDATE `user` SET `name` = 'Bob Updated' WHERE `id` = 2;
-- 事务1未提交，因此锁仍然保持
-- 开启事务2（在另一个会话中）START TRANSACTION;
-- 事务2尝试更新同一行记录，此时会被阻塞，因为事务1已经对该记录加了记录锁UPDATE `user` SET `name` = 'Bob Updated Again' WHERE `id` = 2; -- 此操作会被阻塞，直到事务1提交或回滚
-- 提交事务1（在事务1的会话中）COMMIT;
-- 此时事务2的更新操作才会被执行
```

2. 间隙锁（Gap Lock）示例

```
-- 假设有一个名为test的表，包含id和value字段, value字段是索引字段CREATE TABLE `test` (    `id` INT AUTO_INCREMENT PRIMARY KEY,    `value` INT NOT NULL);-- 插入一些示例数据INSERT INTO `test` (`value`) VALUES (10), (20), (30);
-- 开启事务1START TRANSACTION;
-- 事务1尝试锁定value在15和25之间的间隙SELECT * FROM `test` WHERE `value` BETWEEN 15 AND 25 FOR UPDATE;
-- 事务1未提交，因此锁仍然保持
-- 开启事务2（在另一个会话中）START TRANSACTION;
-- 事务2尝试在锁定的间隙中插入新记录，此时会被阻塞INSERT INTO `test` (`value`) VALUES (22); -- 此操作会被阻塞，直到事务1提交或回滚
-- 提交事务1（在事务1的会话中）COMMIT;
-- 此时事务2的插入操作才会被执行
```

3. 临键锁（Next-Key Lock）示例

```
-- 使用与上面相同的test表
-- 开启事务1START TRANSACTION;
-- 事务1尝试锁定value等于20的记录及其前后的间隙SELECT * FROM `test` WHERE `value` = 20 FOR UPDATE;-- 由于InnoDB在可重复读隔离级别下默认使用临键锁，因此此操作会锁定value为20的记录，以及它前后的间隙
-- 事务1未提交，因此锁仍然保持
-- 开启事务2（在另一个会话中）START TRANSACTION;
-- 事务2尝试更新同一行记录，此时会被阻塞，因为事务1已经对该记录及其间隙加了临键锁UPDATE `test` SET `value` = 21 WHERE `value` = 20; -- 此操作会被阻塞，直到事务1提交或回滚
-- 事务2尝试在锁定的间隙中插入新记录，同样会被阻塞INSERT INTO `test` (`value`) VALUES (19); -- 此操作也会被阻塞，因为锁定了value为20的记录前的间隙
-- 提交事务1（在事务1的会话中）COMMIT;
-- 此时事务2的更新和插入操作才会被执行
```

面试官：说一下MySQL中对于不同的隔离级别，行锁采用的加锁方式有什么不同?

### 隔离级别与行锁加锁方式的关系

MySQL的四种隔离级别分别是：未提交读（Read Uncommitted）、已提交读（Read Committed）、可重复读（Repeatable Read）和串行化（Serializable）。

1. **未提交读（Read Uncommitted）**

- 在这种隔离级别下，事务可以读取未提交的更改，可能会导致“脏读”。
- 锁的使用较少，因此并发性较高，但数据一致性较差。
- 由于允许读取未提交的数据，所以通常不会使用行锁来防止并发问题。

2. **已提交读（Read Committed）**

- 事务只能读取已提交的数据，可以避免脏读。
- 但可能导致“不可重复读”，即在同一事务中对同一数据进行多次读取时，可能会读取到不同的结果。
- 在这种隔离级别下，MySQL会使用行级锁来防止脏读。当事务读取某一行数据时，会对该行加共享锁（S锁），其他事务在读取该行时不会阻塞，但修改该行时会被阻塞。

3. **可重复读（Repeatable Read）**

- 这是MySQL的默认隔离级别。
- 事务在执行过程中，读取的数据是事务开始时的快照，能够保证多次读取返回相同的结果。
- 在这种隔离级别下，MySQL使用多版本并发控制（MVCC）来存储数据的快照，并通过行级锁来防止并发修改。当事务修改某一行数据时，会对该行加排他锁（X锁），其他事务在读取或修改该行时都会被阻塞。
- 此外，为了防止幻读现象（即在一个事务中读取到另一个事务新插入的数据），MySQL的可重复读隔离级别还会使用Next-Key Locks（临键锁），它结合了记录锁（Record Locks）和间隙锁（Gap Locks）。

4. **串行化（Serializable）**

- 这是最高的隔离级别，强制每个事务逐一执行，避免了任何并发问题。
- 在这种隔离级别下，MySQL会对所有读取的数据行加锁，相当于将每个事务完全串行化执行。这会导致并发性极低，但数据一致性最高。

![](https://mmbiz.qpic.cn/mmbiz_png/B220micXXwONVyVldEl27Z24AnpmXh0ibRstgYRSwATibYSiann6b7fv0diap93V4QPaQFGqI2VXrnxwkNYKs3rSnWQ/640?wx_fmt=png&from=appmsg)

面试官：MySQL中的死锁是如何产生的，请给出示例？如何避免和解决死锁问题？

###

### MySQL中的死锁产生原因

MySQL中的死锁是指两个或更多的事务在执行过程中，因相互竞争资源而造成的一种阻塞现象，此时每个事务都在等待其他事务释放它所持有的资源，导致这些事务都无法继续执行。

死锁的产生原因主要包括以下几点：

1. **并发事务冲突**

   如果一个事务已经锁定了资源，而其他事务也试图修改这个资源，那么就可能会产生冲突，导致死锁。
2. **锁定的顺序不一致**

   如果两个事务在锁定资源时采取的顺序不一致，也可能导致死锁。例如，事务A先锁定了资源1，然后试图锁定资源2；而事务B先锁定了资源2，然后试图锁定资源1。这种情况下，事务A会等待事务B释放资源2，而事务B会等待事务A释放资源1，从而形成死锁。
3. **长时间等待资源**

   如果一个事务在等待一个已经被其他事务锁定的资源时，等待时间过长，也可能会产生死锁。
4. **事务尚未完成就请求新的资源**

   在事务尚未完成的情况下，已经锁定的资源不会被释放。如果此时事务再请求新的资源，而该资源已被其他事务锁定，就可能导致死锁。

#### 示例一：锁定的顺序不一致导致死锁

假设有一个名为`orders`的订单表，包含`order_id`、`product_id`和`status`等字段。现在有两个事务A和B，它们分别尝试更新不同订单中的相同产品库存，但操作顺序不同，导致死锁。

```
-- 开启事务ASTART TRANSACTION;
-- 事务A尝试更新订单1中的产品库存UPDATE orders SET status = 'shipped' WHERE order_id = 1 AND product_id = 101;
-- 事务A接着尝试更新订单2中的相同产品库存（此时订单2可能尚未被锁定）UPDATE orders SET status = 'shipped' WHERE order_id = 2 AND product_id = 101;
```

```
-- 开启事务BSTART TRANSACTION;
-- 事务B尝试更新订单2中的产品库存（与事务A中的第二个UPDATE操作相同）UPDATE orders SET status = 'shipped' WHERE order_id = 2 AND product_id = 101;
-- 事务B接着尝试更新订单1中的相同产品库存（此时订单1可能已被事务A锁定）UPDATE orders SET status = 'shipped' WHERE order_id = 1 AND product_id = 101;
```

示例二：多事务间隙锁+insert导致死锁

假设有下面这一张表:

|  |  |  |
| --- | --- | --- |
| id（主键索引) | no(非主键索引) | name |
| 1 | 1001 | 小明 |
| 2 | 1002 | 小李 |
| 3 | 1003 | 小华 |
| 4 | 1004 | 小黄 |



事务A首先开启了，执行一条select...for update这样的语句,因为记录的最大值为1004，1007不在这一个范围当中。此时，事务A对于表当中no范围为(1004,+∞)的no索引加上了一把锁间隙锁。



之后事务B开启了，因为no值为1008的记录，不在范围(1004,+∞)的范围之内，事务B也会加一个间隙锁，范围是(1004,+∞)。由于间隙锁之间是互容的，因此事务B在执行select语句的时候，不会发生阻塞。



之后事务A执行了一条插入的索引为1007的数值。



这里需要了解，Insert语句在正常执行的时候，是不会生成锁结构的，它是靠聚簇索引自带的一个被称为trx_id的字段作为隐式锁来保护记录的。只有在指定情况下，才会把隐式锁转化为显示锁，也就是真正加锁的过程。

 

举两个例子来说明隐式锁转换为显式锁的场景:

       (1) 范围(a,b)内加有间隙锁，当有一条记录在范围(a,b)之内插入记录的时候，就会转化为显式锁。

       (2)如果insert语句插入的记录和已有的记录之间出现了主键，也无法插入。



因此事务A执行一条插入sql时就一定要等待到事务B释放锁，才可以继续执行。



最后事务B执行了一条插入的索引值为1008的sql语句。但是由于事务A对于(1004,+∞)的范围加锁了，因此事务B一定需要等待到事务A释放锁，才可以继续执行。



至此，AB事务相互等待对方释放锁，死锁形成。


### 避免和解决死锁问题的方法

为了避免和解决MySQL中的死锁问题，可以采取以下几种方法：

**设****置适****当****的事务隔****离级别**：

事务隔离级别是控制并发访问时数据的一致性与并发性之间的平衡点。在MySQL中，可以通过设置适当的事务隔离级别（如READ COMMITTED或更高）来防止死锁。

拆分事务

将事务拆分为较小的单元，可以减少锁的持有时间，从而降低死锁的发生概率。

设置死锁超时时间

MySQL提供了设置死锁超时时间的参数（如innodb_lock_wait_timeout），可以控制等待锁的超时时间。当超过设定的时间后，将会报出死锁错误，事务会自动回滚。

手动解决死锁问题

当发生死锁问题时，可以使用MySQL提供的命令（如SHOW PROCESS LIST）查看当前正在运行的所有进程，并通过KILL命令强制终止死锁进程。

### 死锁日志分析

**检查MySQL是否已开启死锁日志**

1. 可以通过执行`SHOW VARIABLES LIKE 'log_error';`命令来检查MySQL的错误日志路径。确保MySQL的错误日志功能是开启的，因为死锁信息会自动记录在MySQL的错误日志中。
2. **通过SHOW ENGINE INNODB STATUS获取死锁信息**
4. 在事务发生死锁时，可以执行`SHOW ENGINE INNODB STATUSG`命令。

   输出中的`LATEST DETECTED DEADLOCK`部分会显示最近一次死锁的详细信息，包括哪些事务参与了死锁、被锁住的SQL语句、加锁的表和行等。
6. 关于死锁日志的具体分析可以参考文章：
7. https://blog.csdn.net/lkforce/article/details/108781691

###

---

欢迎在评论区留言表达看法和问题，阿沛会一一作出回复。

如果本文对大家有帮助，麻烦大家动动小手点个免费的“赞”或“在看”，大家的鼓励就是阿沛持续更新的动力~

商务合作或加技术交流群欢迎联系小助手~

![](https://mmbiz.qpic.cn/mmbiz_png/B220micXXwONSEXlt0ALPwia9UP17IBDD50esjpEsylJmg78Q5bF8SiaibnDusFqdRxxib2pQRHKO8odwj318tHKkJw/640?wx_fmt=png&from=appmsg)

**-- 往期精彩 --**

**[吊打面试官系列——万字长文MySQL篇面试题总结（二）](https://mp.weixin.qq.com/s?__biz=MzUxMjczODMyMA==&mid=2247484468&idx=1&sn=3f73d1c3c51986d9d696dca9078e78eb&scene=21#wechat_redirect)**

**[面试题：请说说看Mysql的Buffer Pool如何影响数据库性能？说说Change buffer和它的作用？](https://mp.weixin.qq.com/s?__biz=MzUxMjczODMyMA==&mid=2247485404&idx=1&sn=0b594af82b00018c331629350fef500e&scene=21#wechat_redirect)**

**[华为一面：说说看MySQL的索引类型及其作用？什么是聚簇索引和非聚簇索引？索引失效的情况有哪些？说说看回表、覆盖索引和索引下推？](https://mp.weixin.qq.com/s?__biz=MzUxMjczODMyMA==&mid=2247485305&idx=1&sn=e86a2d16cf5cd02c935f6175ad9e4174&scene=21#wechat_redirect)**

### [MySQL怎么运行的系列（十一）快照读、锁定读、半一致性读 和 加锁语句分析](https://mp.weixin.qq.com/s?__biz=MzUxMjczODMyMA==&mid=2247483816&idx=1&sn=431ba859c0158d82fa4f4b4e0c373226&scene=21#wechat_redirect)

### [Redis面试题：什么是Redis的大key和热key，它们会造成什么问题，如何解决？](https://mp.weixin.qq.com/s?__biz=MzUxMjczODMyMA==&mid=2247485761&idx=1&sn=2c02dc7d4817cdd98d887f690bc64768&scene=21#wechat_redirect)

---

> 原文链接：http://mp.weixin.qq.com/s?__biz=MzUxMjczODMyMA==&mid=2247485944&idx=1&sn=1e4e1a0517494289ce2b4994ac33fc49&chksm=f86892264a979d9439f4b6f77bb75b667266d62520f47358efb12d88302d6417d8f3c9dde42a&mpshare=1&scene=24&srcid=0103MDI8rQ15d4i7u3H3MwmQ&sharer_shareinfo=e60f3f62550363955d8df1e434053e32&sharer_shareinfo_first=e60f3f62550363955d8df1e434053e32#rd
> 收藏时间：2025-01-03 17:06:04 | 来源（wxid）：wxid_y9ar5ahmtw1c22 | 收藏项 ID：694

