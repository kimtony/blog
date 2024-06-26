### 一、sql调优


### 解释
* 一级索引（主键索引/聚簇索引）
```
id 列是主键，它被定义为聚簇索引。
数据行会根据 id 列的顺序存储在磁盘上。
```
* 二级索引（非聚簇索引）
```
email 列上有一个非聚簇索引 idx_email。
idx_email 的叶子节点存储 email 和指向对应数据行的指针（即 id）。
查询 email 列时，首先通过 idx_email 索引找到对应的 id，然后通过 id 在聚簇索引中找到实际的数据行
```
* 查询非索引列时需要进行一次额外的回表操作（查找数据行）。

#### 什么是最左匹配原则
```
最左匹配原则指的是数据库查询优化器在使用复合索引时，会优先利用索引定义中最左侧的列进行匹配。只有当查询条件中包含索引定义中最左侧的一列时，索引才能被有效使用。
```
```
创建复合索引
CREATE INDEX idx_user ON users (last_name, first_name, age);

使用最左列
SELECT * FROM users WHERE last_name = 'Smith';

使用最左两列
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';

使用所有列
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John' AND age = 30;

不遵循最左匹配原则的查询
SELECT * FROM users WHERE first_name = 'John';
```



#### 优化思路
* 1.白盒分析
```
此阶段我们可以根据自己已有的知识积累和经验，猜想sql可能慢的原因。
```
* 2.执行计划解读
```
通过explain结果，解读并模拟出mysql服务器对sql的真实执行过程。
```
* 3.瓶颈点确定
```
白盒分析+执行计划解读之后，基本可以确定sql可能慢的原因。
```
* 4.对症下药
```
找到瓶颈点后，逐个击破。
接下来，我就通过两个实例（单表查询和连接查询）与大家一起交流下调优的一些心得。
以下的调优过程主要是站在sql层面进行优化。并非站在整个系统方面，如分表、切换存储媒介等
```




### 常用方法
```
加索引 索引遵循最左匹配的原则、
select时指明具体字段、
不建议使用函数、
超过3张表不建议join
尽量使用批量插入、更新和删除操作，减少单次操作的开销。
```

### 单表sql查询
* 单表查询sql的执行，大体上可以分为两步。"二级索引find"和"回表select"。
```
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    age INT,
    INDEX (email)
);
SELECT * FROM users WHERE email = 'example@example.com';

【执行过程】
在 email 列的索引中查找 example@example.com。
找到索引条目后，获取指向数据行的指针（主键值 id）。
使用主键值 id 回表，从 users 表中读取完整的行数据。


【优化策略】
1、覆盖索引：如果查询只涉及索引列，可以避免回表。例如：
SELECT email FROM users WHERE email = 'example@example.com';

2、复合索引：如果查询条件涉及多个列，可以创建复合索引来覆盖这些列，减少回表次数。例如
CREATE INDEX idx_users_email_age ON users(email, age);

```


### 多表sql查询
* 多表链接连接本身是一个求笛卡尔积的过程。
```
SELECT * FROM t1, t2 WHERE t1.c1 > 1 AND t1.c1 = t2.c1 AND t2.c2 < 'd';

单表查询条件：t1.c1 > 1和 t2.c2 < 'd'

多表查询条件：t1.c1 = t2.c1
```
* 连接过程
 * 1.确定驱动表，假定t1表为驱动表
 * 2.针对步骤1所匹配到的记录，分别去被驱动表t2中查找数据。
 ```
 因为从驱动表t1中查找到了2条记录，因此会对被驱动表t2执行两次单表查询。涉及多表查询的条件t1.c1 = t2.c1此时登场。
 ```
 * 3.将步骤2查询到的结果合并即为最终结果
 ```
 对于连接查询来说，驱动表只会访问一次，被驱动表将会访问多次，具体由驱动表执行单表查询后的结果集中的记录条数决定。
以上属于比较直接的方法，遍历步骤1的结果，一次一次去对被驱动表进行查询（虽然查询被驱动表时可以利用被驱动表的索引加快），称之为嵌套循环连接（Nested-Loop Join）。


基于嵌套循环连接，Mysql对其进行了横向优化，提出了join buffer的概念。

利用join buffer可以一次将从驱动表得到的多条记录与被驱动表进行连接，这样便可减少被驱动表的访问次数。这种方式称之为基于块的嵌套连接（Block Nested-Loop Join）。
 ```

### 
```
1.sql的优化，如果可能，回表的消耗需要降到最低。
2.在进行连表查询时，如果需要排序操作，尽量按照驱动表的索引字段进行排序，因为其天然有序，可以避免大数据量下，mysql创建临时表去排序，这是非常耗资源的。
3.当遇到慢sql后，执行计划是我们进行优化的切入点，我们能清楚了解整个sql的具体执行过程，就能从中找到瓶颈点，从而对症下药。
4.彼时的调优可以解决彼时的问题，随着业务的发展，数据量会上来，数据分布也会变得更加不均匀，待到那时，可能会再次面临调优。
5.在写sql时可以遵从一些成熟的经验总结，提前避免一些问题。但是系统总是向前发展的，不会说可以用一条sql应对各种场景，调优是个持续的过程。
6.细节很重要，单条sql执行时间过慢的原因可能是多个细节累加造成。10个0.1s就是1s。
```
