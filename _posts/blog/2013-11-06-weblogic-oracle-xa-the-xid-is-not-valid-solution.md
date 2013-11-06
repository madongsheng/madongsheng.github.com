---
layout: post
title: Weblogic Oracle使用XA驱动XID is not valid问题
description: Weblogic Oracle的XA驱动使用分布式事务时会遇到XID is not valid的情况，但这里的问题和以前的不太一样！
category: blog
---

Weblogic下用EJB实现分布式事务如何时报下面的错：

>java.sql.SQLException: Unexpected exception while enlisting XAConnection java.sql.SQLException: XA error: XAResource.XAER_NOTA start() failed
 on resource 'CTDataSource': XAER_NOTA: The XID is not valid

除了是事务超时，要调整事务超时时间外，还可能是更蛋疼的原因：**两个Weblogic Domain中配置的DataSource同名！**是的，没错，两个Domain中的数据源
如果同名例如都是：CTDataSource，也会报上面的错误，而且这个错误不太容易找出来。Weblogic在使生成XID的时候使用的是DataSource的名（weblogic中的名字不是jndi名)，
我想，更好的方式是使用名字+随机生成的方式吧。



