## 事务概述



### 1. 简介

 事务是并发控制的单元，是用户定义的一个操作序列。是一个不可分割的工作单位。

- 自动提交事务： 每条单独的语句都是一个事务，每个语句后都隐藏一个commit
- 显示事务：以 begin transaction 显示开始，以commit 或者rollback结束
- 隐式事务 ： 自动开始，但需要手动显示提交。

1. 其他概念
   1. 保存点（savePoint）：设置了保存点后，可以rollback到改保存点，而不是rollback整个事务。
   2. Mysql 的表类型为Innodb  才支持事务。

### 2. 事务ACID原则

1. 原子性 （atomicity）：事务在执行的过程中所做的操作要嘛全部成功，要嘛全部无效
2. 一致性（consistency）：再事务执行失败时，所有被该事务影响的数据都应该恢复事务执行前的状态。
3. 隔离性（isolation）：事务执行过程中对数据的修改再提交前，对其他事务是不可见的。
4. 持久性（durability）：事务一旦提交，事务的操作便永久的保存在数据库中，即使执行回滚操作也无效。

### 2. java事务的类型

1. jdbc事务：Java提供`java.sql.Connection `接口，提供两种事务模式：自动提交和手动提交，事务不能夸多个数据库
   1. Connection类中提供的控制事务的方法
      - `setAutoCommit（boolean autoCommit）；`: 设置是否自动提交事务
      - `commit（）；` 提交事务
      - `rollback（）；`撤销事务
      - `setTransactionIsolation();`:设置隔离级别
2. JTA（Java transaction API）事务：允许分布式事务处理（事务可夸多个数据库）。
3. 容器事务：J2EE 应用服务器提供的事务管理，基于JTA。

### 3.事务并发处理可能出现的问题 

1. 脏读：一个事务读取了另一个尚未提交的数据。
2. 不可重复读： 一个事务的操作导致另一个事务前后两次读取到不同的数据。
3. 幻读： 一个事务的操作导致另一个事务前后两次查询的结果 **数据量** 不同


### 4. 事务的隔离级

-  作用： 为解决事务并发处理可能出现的问题

1. TRANSACTION_NONE : jdbc驱动不支持事务
2. TRANSACTION_READ_UNCOMMITTED : 允许脏读，不可重复读，幻读
3. TRANSACTION_READ_COMMITED: 禁止脏读，但允许不可重复读和幻读
4. TRANSACTION_REPEATABLE_READ: 禁止脏读和不可重复读，允许幻读
5. TRANSACTION_SERIALIZABLE : 禁止脏读，不可重复读和幻读

### 5.事务传播特性

一. 概述 ： 事务的传播特性是由于事务的嵌套引起的。以下前六种特性由TransactionDefinition接口定义。最后一个是由Spring所提供的特殊变量。

``` 
ServiceA{
  void methodA(){
    ServiceB.methodB();
  }
}

SreviceB{
  void methodB(){ }
}

以上为事务的嵌套，ServiceA.methodA 为外部事务，ServiceB.methodB为内部事务
```

1. **PROPAGATION_REQUIRED ** : 如果存在一个事务，则支持当前事务，如果没有事务则开启		

   1. 如果外部事务已启用，则ServiceA.methodA内部就不会启用新的事务。ServiceB.methodB就会公用外部事务。ServiceA.methodA和ServiceB.methodB 就会作为一个整体。无论哪里出现异常都会一起回滚
   2. 如果ServiceA.methodA没有事务，ServiceB.methodB 就会分配一个新事务，该事务的作用范围仅限于ServiceB.methodB

2. **PROPAGATION_SUPPORTS **  : 如果存在一个事务，支持当前事务。如果没有事务，则非事务的执行

3. **PROPAGATION_MANDATORY**  : 如果已经存在一个事务，支持当前事务，如果没有一个活动的事务，则抛出异常。即必须在一个事务中运行

4. **PROPAGATION_REQUIRES_NEW ** :  总是开启一个新的事务，如果一个事务已存在，则将这个存在的事务挂起。启动一个新的,不依赖于环境的 "内部" 事务. 这个事务将被完全 commited 或 rolled back 而不依赖于外部事务, 它拥有自己的隔离范围, 自己的锁, 等等. 当内部事务开始执行时, 外部事务将被挂起, 内务事务结束时, 外部事务将继续执行. 内部事务与外部事务直接互相独立

5. **PROPAGATION_NOT_SUPPORTED ** : 总是非事务的执行，并挂起任何存在的事务

6. **PROPAGATION_NEVER ** : 总是非事务的执行，如果存在一个活动的事务，则抛出异常

   ​	如果ServiceA.methodA的事务级别是PROPAGATION_REQUIRED ,而ServiceB.methodB的事务级别是PROPAGATION_NEVER,那么ServiceB.methodB就要抛出异常了

7. **PROPAGATION_NESTED**  : 如果一个活动的事务存在，则运行在一个嵌套的事务中，如果没有活动事务，则按TransactionDefinition.PROPAGATION_REQUIRED执行。内部事务开始执行时，取得一个保存点，如果内部事务失败，回滚至保存点。内部事务是外部事务的一部分，只有外部事务结束它才能提交。