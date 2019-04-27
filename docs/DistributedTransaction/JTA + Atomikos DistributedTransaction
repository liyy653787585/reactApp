最最基础的分布式事务，单系统跨多库的分布式事务，采用的是JTA + Druid多数据源 + Atomikos，基于这三者整合而成的分布式事务的管理，底层的思想其实就是2PC的原理

整合在spring boot + druid生产级环境中的一套单系统跨多库的分布式事务实践，使用的JTA事务管理器，加上Atomikos分布式事务的框架，Atomikos框架主要是提供了支持分布式事务的DataSource数据源的一个支持JTA主要是提供了事务管理器，也就是分布式事务流程管控的这么一套机制

1 connection 动态代理的构建（核心重点）
getConnection()方法返回的connection 其实是一个实现了Connection接口的动态代理，其对应的handle是AtomikosConnectionProxy，当connection 动态代理被调用的时候（执行sql时），都会由它来拦截来处理，执行xa-start、xa-end指令

对每个库都发送了XA START指令，同时对每个库都使用prepareStatement()方法定义好了每个库要执行哪些SQL语句，下一步就会去执行整个事务的这么一个提交，其实就会走2PC的一套逻辑了

对每个库都执行XA END指令，然后对每个库都执行XA PREPARE指令，2PC的阶段一，如果说有人的prepare返回消息不是ok，那么就全量回滚，就是发送XA ROLLBACK指令，如果说所有人的返回消息都是ok，那么就对每个库发送XA COMMIT指令


站在比较高的一个角度，抽象的角度，来思考一下JTA + Atomikos实现XA分布式事务框架的架构设计的一个特点，基于Spring原生transaction事务控制的一个过程

JTA，其实是负责了Atomikos框架的一个指挥和调度，让他按照事务的基本步骤来走
Atomikos框架相当于是DTP模型里的TM，跟各个MySQL（RM）通信，XA START、XA END、XA COMMIT、XA ROLLBACK等符合XA规范的一套接口，调用

2 注意需要与spring的@transactions注解源码结合
创建分布式事物、事物commit、事物rollback都是由@transactions源码触发的，很重要！
