---
title: "MySql事务"
date: 2019-02-13T15:00:01+08:00
draft: false
tags: 
  - MySql
---

**事务的四个属性**

​	事务是必须满足4个条件（ACID）：原子性（**A**tomicity，或称不可分割性）、一致性（**C**onsistency）、隔离性（**I**solation，又称独立性）、持久性（**D**urability）

- **原子性：**一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- **一致性：**在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
- **隔离性：**数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- **持久性：**事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

**并发事务带来的问题**

+ **脏读(Drity Read)：**某个事务已更新一份数据，另一个事务在此时读取了同一份数据，由于某些原因，前一个RollBack了操作，则后一个事务所读取的数据就会是不正确的。
+ **不可重复读(Non-repeatable read)：**在一个事务的两次查询之中数据不一致，这可能是两次查询过程中间插入了一个事务更新了原有的数据。（例如银行想查询A帐户余额，第一次查询A帐户为200元，此时A向帐户内存了100元并提交了，银行接着又进行了一次查询，此时A帐户为300元了。银行两次查询不一致，可能就会很困惑，不知道哪次查询是准的。）
+ **幻读(Phantom Read)：**在一个事务的两次查询中数据笔数不一致，例如有一个事务查询了几列(Row)数据，而另一个事务却在此时插入了新的几列数据，先前的事务在接下来的查询中，就会发现有几列数据是它先前所没有的。
+ **更新丢失：** 两个不同的事务（或者Java程序线程）在某一时刻对同一数据进行读取后，先后进行修改。导致第一次操作数据丢失。

**事务的隔离级别**

​	在数据库操作中，为了有效保证并发读取数据的正确性，提出的事务隔离级别，在标准SQL规范中，定义了4个事务隔离级别，不同的隔离级别对事务的处理不同。

+ **读未提交（Read uncommitted）：**如果一个事务已经开始写数据，则另外一个事务则不允许同时进行写操作，但允许其他事务读此行数据。（会引发脏读取、不可重复读和虚读，但避免了更新丢失。）

  ```mysql
  ## 建表
  CREATE TABLE `account` (
    `name` varchar(20) DEFAULT NULL,
    `money` decimal(10,2) DEFAULT NULL
  )
  ## 插入数据
  INSERT INTO account (NAME, money) VALUES('A', 500);
  INSERT INTO account (NAME, money) VALUES('B', 500);
  ```

  eg:

  ```mysql
  ## A窗口
  SET GLOBAL TRANSACTION ISOLATION  LEVEL READ UNCOMMITTED;##设置数据库隔离级别为Readuncommitted(读未提交)
  start transaction;##开启事务
  select * from account where name = 'A';##(500)查询A账户中现有的钱，转到B窗口进行操作
  select * from account where name = 'A';##(600)发现a多了100元，这时候A读到了B未提交的数据（脏读）
  ## B窗口
  start transaction;##开启事务
  update account set money=money+100 where name='A';##不要提交，转到A窗口查询
  ```

+ **读提交（Read Committed）:**读取数据的事务允许其他事务继续访问该行数据，但是未提交的写事务将会禁止其他事务访问该行。

  eg:

  ```mysql
  ## A窗口
  SET GLOBAL TRANSACTION ISOLATION  LEVEL READ COMMITTED;
  START TRANSACTION;
  SELECT * FROM account WHERE NAME = 'A';##发现a帐户是500元，转到b窗口
  SELECT * FROM account WHERE NAME = 'A';##b窗口提交事物后发现a帐户多了100,这  时候，a读到了别的事务提交的数据，两次读取a帐户读到的是不同的结果（不可重复读）

  ## B窗口
  START TRANSACTION;
  UPDATE account SET money=money+100 WHERE NAME='A';##不提交转到窗口a查询a账户金额为500（避免了脏读）
  COMMIT;##转到a窗口
  ```

+ **可重复读取（Repeatable Read）(mysql默认级别)：**读取数据的事务将会禁止写事务（但允许读事务，不管事务b是否提交事务a查询结果保持不变，基于快照读），写事务则禁止任何其他事务。

  eg:

  ```mysql
  ## A窗口
  SET SESSION  TRANSACTION ISOLATION  LEVEL REPEATABLE READ;
  START TRANSACTION;
  SELECT * FROM account;##发现表有2条记录，转到b窗口
  SELECT * FROM account;##b事务提交后查询任为2条记录mysql的RR（可重复读）一并解决了幻读的问题
  COMMIT;
  SELECT * FROM account;##a事务提交后查询为3条记录
  
  ## B窗口
  START TRANSACTION;
  INSERT INTO account(NAME,money) VALUES('C',500);
  COMMIT;##转到a窗口
  ```

+ **序列化（Serializable）：**提供严格的事务隔离。它要求事务序列化执行，事务只能一个接着一个地执行，不能并发执行。

  eg:

  ```mysql
  ## A窗口
  set transaction isolation level Serializable;
  start transaction;
  select * from account;##转到b窗口
   
  ## B窗口
  start transaction;
  insert into account(name,money) values('C',500);##发现不能插入，只能等待a结束事务才能插入
  
  ```
