## Spring事务

事务的特性是什么？

事务的概念？

多个操作组成1个单元，要么同时成功，要么同时失败

@Transational干了什么事？

事务的隔离级别



@Transational

@Transational(propagation=Propagation.Required)

如果当前线程存在事务，则使用当前事务,如果不存在则新建一个事务



@Transational(propagation=Propagation.Required)

如果当前线程存在事务，则挂起当前事务，并且新建一个事务继续执行，新建事务完成之后，唤醒挂起的时候，继续执行。

如果当前线程不存在事务，则新创建一个事务。





