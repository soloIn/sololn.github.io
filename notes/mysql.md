# SQL 语法

## 数据类型

- 整型  -- tinyint (1)    smallint (2)  mediumint(3)  int (4)  bigint(8)   , int(11) 只是规定了显示的长度，对于存储没有意义
- 浮点  --  float   double  decimal  
- 字符串 -- char 定长  varchar    char 效率高，varchar 空间利用率高。  char 会删除右侧空格，varchar 会保留所有空格
- 日期和时间  -- DATETIME  TIMESTAMP(时间戳) ，插入时没有指定TIMESTAMP 会自动插入当前时间
- 

## 过滤

- where   sql 限制条件，满足限制条件下select ，即先where 再select ，不支持聚合函数
- having  过滤条件，先查询在过滤，支持聚合函数

## 链接

- 笛卡尔积 -- select * from A ,B 即所有数据
- 等值链接(内链接) --  inner jion B where  .. on ..
- 自然连接(特殊的等值连接)  -- nature jion 
  - 比较的部分必须是相同的属性名(组)
  - 会把重复的属性列去掉



## 存储过程

SQL批量处理

优点

- 预编译，效率高
- 复用

定义

```sql
delimiter //  -- 定义结束符

create procedure myprocedure(in p_int int, out ret int，inout p_int2 int )  -- 三种参数
    begin  -- 存储过程开始
        declare y int; 定义变量
        select sum(col1)
        from mytable
        into y;  -- 给变量复制 select into 或者 set
        select y*y into ret;
    end //  -- 存储过程结束

delimiter ; -- 将语句的结束符恢复为分号
set @p = 1  --设置变量
call myprocedure(p,p_out,p_inout);
select @p -- 查看变量
```



## 游标

游标可以对结果集集进行遍历，可以定位到行

```sql
declare mycursor cursor for select id
open mycursor
repeat
	fetch mycursor into ret
	select ret;
until "4" end repeat;
close mycursor
```





## 触发器

触发器会在表执行 UPDATE、INSERT、DELETE 三个关键字时执行

- UPDATE  包含一个 NEW 、OLD 表
- INSERT 包含一个NEW 虚拟表
- DELETE 包含一个OLD 虚拟表

作用

- 写入表之前，进行数据校验、数据转换、净化数据等
- 写入表之后，审计跟踪

```sql
delimiter 自定义结束符号
create trigger 触发器名字 触发时间(before or after) 触发事件(insert or update or delete ) on 表 for each row
begin
    -- 触发器内容主体，每行用分号结尾
end
自定义的结束符合
delimiter ;
```

## 事务管理

一组SQL 要么全部执行完，失败回滚

不能回退 select create alter 语句

MYSQL默认隐式提交，即每一句SQL当作事务来提交。start transaction 关闭 隐式提交，commit or rollback 提交显示提交，恢复隐式提交

```sql
// 设置了保留点
START TRANSACTION
// ...
SAVEPOINT delete1
// ...
ROLLBACK TO delete1
// ...
COMMIT
```



## 权限管理

权限放在mysql数据库中

```sql
-- 赋权
GRANT SELECT, INSERT ON mydatabase.* TO myuser;
-- 废权
REVOKE SELECT, INSERT ON mydatabase.* FROM myuser;

```



# 数据库系统原理

## 事务

事务指满足 ACID 特性的一组操作

ACID

- #### atomicity (原子性)

- consistency (一致性)  执行事务前后，数据库都保持一致性状态，一致性状态-- 事务前后数据都要满足预定的约束，比如银行转账的例子

- Isolation (隔离性)

- Durability (持久性)

## 并发一致性

并发环境中，事务的隔离性很难保证

1. 修改丢失

   ![](./img/丢失更改.png)

2. 读取了其他事务的修改，当该修改又撤销了

   ![](./img/读脏数据.png)

3. 两次读取到的结果不同

   ![](./img/不可重复读.png)

4. 幻读

   幻读针对 “读-写”，比如插入之间检测是否有结果，结果这是被其他事务插入了，导致当前事务检测没有，插入却错误。

## 封锁

MYSQL 提供了两种锁粒度，表锁、行锁

锁类型

- 互斥锁 -- 写锁
- 共享锁 -- 读锁

### 意向锁

事务A 锁住了一个行写锁，事务B申请表读锁，数据库需要判断是否有冲突

1. 判断有没有别的事务加了表写锁
2. 判断每一行是否存在写锁  -- 这一步很耗时，意向锁就是为了解决这个问题

意向锁 IX/IS都是表锁，它表示某个事务想在数据的某一行加X/S锁

- 事务在获得行S锁之前，必须先获得的IS或者更强的锁
- 事务在获得行X锁之前，必须先获得IX锁

作用

```
当要添加表锁时，有意向锁说明存在行级锁，不在需要遍历行来判断是否有行锁
如果想添加行锁，则不受影响
总结，意向锁是为了更好的支持多粒度的锁
```



### 封锁协议

1. 一级封锁协议

   事务T修改数据加X锁，直到T完成后才释放锁，可以解决修改丢失的问题，因为不会有两个事务同时修改数据

2. 二级封锁协议

   事务B 读取数据是加S锁，读完立马释放S锁，因为需要等待事务T归坏锁了，B才能加S锁，所以可以避免读到脏数据	

3. 三级封锁协议

   事务B 读取数据是加S锁，事务完成后才释放S锁。解决不可重复读的问题。因为加S锁有，其他事务不能加X锁



### 两段锁协议

指加锁和解锁分为两个阶段进行，是串行化调度的充分条件

串行化调度:事务并发执行的结果与某个串行执行的结果相同。



## 隔离级别

1. 未提交读 -- 事务修改了但未提交的时候，其他事务依然可以读
2. 提交读  即事务修改数据并提交对其他事务是不可见的
3. 可重复读 提交读不能保证可重复度，因为可能又有别的事务修改了数据
4. 可串行化 强行事务串行执行，串行执行就没有数据一致性的问题

## 多版本并发控制

Multi-Version Concurrent Control -- MVCC 多版本并发控制是 InnoDB 实现隔离级别的一种具体方式

1. 基本思想

   MVCC 中，事务修改数据会给数据行生成一个版本快照，规定其他事务只能读取已经提交的快照。这样读写就没有互斥关系，即写不会导致读阻塞。这样可以提高并发能力

2. 版本号

   1. 系统版本号，每开始一个事务，版本号就加一
   2. 事务版本号，事务开始时，系统的版本号

3. UNDO 日志

   MVCC 的多版本指的时多个版本快照，快照存储在undo 日志中，日志通过 ROLL-PTR 把改行所有的版本快照连接起来

   ![](./img/MVCC.png)

   例， 由于默认隐式事务，所以会产生三个事务。快照如图所时。快照包含TRX_ID 事务版本号 ，DEL 是否被删除

   ```sql
   INSERT INTO t(id, x) VALUES(1, "a");
   UPDATE t SET x="b" WHERE id=1;
   UPDATE t SET x="c" WHERE id=1;
   ```

   INSERT UPDATE DELETE 操作会生成 undo 日志

4. ReadView

   包含当前系统未提交的事务TRX_ID(事务版本号标记)，并拥有两个指 TRX_ID_MIN,TRX_ID_MAX ，当进行select 操作时，根据三个数据进行判断 -- 快照是否可用

   - TRX_ID < TRX_ID_MIN  说明当前快照是所有事务启动之前的部分，可以使用
   - TRX_ID > TRX_ID_MAX 说明当前快照是在所有事务之后，不可使用
   - TRX_ID_MIN < TRX_ID < TRX_ID_MAX 分情况考虑
     - 如果TRX_ID 出现在 TRX_IDs 列表中，说明事务没有提交，如果没有出现，说明提交了。提交读的级别是可以的

   如果快照不可使用的化，需要 undo 日志回滚到前一个快照，因为 undo 日志包含所有快照，而 ReadView 只包含未提交的事务

5. 快照读 当前读

   快照读读取的是快照的数据，不会加锁，并发能力强

   ```sql
   select * from tableName
   ```

   当前读会加锁

   ```sql
   select * from tableName lock in shard mode  -- 加S 锁
   select * from tableName for update  -- 加X锁
   ```

   

## Next-Key locks

MVCC +  NEXT-KEY lock 可以在RR级别解决幻读问题，它包含

- record lock 锁定记录上的索引，但不包含所有本身
- Gap lock  锁定索引的间隙

```sql
-- 这时候就不能在 16 -- 20 之间插入数据了
select * from myTable where id between 16 and 20 lock in share mode  
```





## 关系型数据库中的设计理论



## ER 图





# MYSQL

## 索引

### B + 树原理

1. 与红黑树比较

   使用B+ 树来作为索引结构是因为访问磁盘有更好的性能

   - 树高低，磁盘寻道的时间与树高成正比
   - 磁盘预读特性

### MYSQL 索引

MYSQL 索引是在存储引擎层实现的

1. B + 索引

   大多数MYSQL 的存储引擎的默认索引

   因为B + 的有序性，所以除了查找，还方便 排序、分组

2. 聚簇索引 与 辅助索引

   一张表只有一个聚簇索引(一般是主键)，聚簇索引包含索引和数据，也就是用主键构建B+ 树，然后叶子节点存放DATA

   辅助索引(二级索引)， data 域存放主键值，所以它需要进行二次搜索-- 首先查到主键值，然后再去主索引搜索。

   InnoDB 使用聚簇索引 ， MyIsam 使用非聚簇索引

3. 哈希索引

   查找快，但不能分组与排序，也不能范围查找

4. 全文索引

   通过关键字查找，它记录着关键字与其对应文档的映射

   ```sql
   select * from fulltext_test 
       where match(content,tag) against('xxx xxx')
   ```

   

### 索引优化

1. 独立的列

   使用索引时，索引不能时表达式的部分，不能时函数的参数

2. 多列索引比单列索引块

3. 多列索引顺序 -- 选择性更强的索引放前面

   选择性更强是指： 不重复的索引值的数量与记录总数的比值，越大选择性越强。也就是值重复的少，比如ID

4. 前缀索引

   对于BLOB、TEXT、 VARCHAR 类型的列，必须使用前缀索引，索引长度考虑选择性取

   ```sql
   alter table city_demo add key (desc(6));  --给desc 列添加长度为6 的前缀索引
   select * from city_demo where desc like "aaaaaa%"  查询
   ```

5. 覆盖索引

   把查询的列的值包含到索引中，比如给查询列添加索引

   - 若二级索引能覆盖查询，那么就不用访问主索引

### 索引的优点

1. 优点
   1. 减少扫描行
   2. 避免排序和分组，因为B+树是有序的



### 查询性能优化

#### EXPLAIN

关注三个字段

- select_type -- 简单查询，联合查询，子查询
- key -- 使用的索引
- rows -- 扫描的行数
- type -- 访问类型
  - all 全盘扫描
  - index 遍历索引树 , 索引是有序的，所以如果需要排序，比all 效率高，否则不一定
  - range 范围索引遍历
  - ref 条件列使用了索引，index 没有使用条件，而ref 使用了条件，并且条件列是索引
  - eq_ref 和ref 的区别是索引是唯一键
  - const

#### 优化数据访问

1. 只返回必要的列
2. 只返回必要的行
3. 使用缓存
4. 减少服务器扫描的行 -- 索引



#### 重构查询方式

1. 切分大查询，避免阻塞其他重要的小查询
2. 分解关联查询
   1. 让缓存更容易中
   2. 减少锁占用时间
   3. 数据库扩展更灵活，比如要移动表的时候



## 存储引擎

- myisam 不支持事务，表锁，不支持外键

## 分表

1. 水平切分(sharding) -- 数据量太大
2. 垂直切分  -- 关系密集程度



#### sharding 策略

1. hash取膜：hash(key) % N
2. 范围：ID范围或是时间范围

问题 

1. 事务 -- 分布式事务
2. ID唯一性 -- 全局id，每个分片设定id范围



### 主从复制

![](./img/master-slave.png)



涉及到到三个线程

- binlog ： 将主数据库的更改写入二进制文件
- I/O：从主服务器读取二进制文件并写入从服务器的中继日志(Relay log)
- SQL : 读取Relay log 文件，解析数据并在从服务器上重放

   

   



