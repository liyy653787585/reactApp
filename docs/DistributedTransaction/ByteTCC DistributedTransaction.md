# 分布式事物框架ByteTCC（重点）
## 系统启动时创建动态代理（核心重点）
### connection动态代理
扫描到bytetcc自己的LocalXADataSource，针对LocalXADataSource创建了一个动态代理，动态代理的InvocationHandler是ManagedConnectionFactoryHandler
### FeignInvocationHandler动态代理
CompensableFeignHandler，bytetcc自己搞了一个CompensableFeignHandler（含有FeignInvocationHandler的引用）来封装了这个feign自己原生的InvocationHandler，这样的话，所有对@FeignClient接口的调用，首先都会走到bytetcc里面去

## @Transactional注解控制整个分布式事物执行流程的执行
你只要加了这个@Transactional注解，就可以看到TransactionInterceptor的东西就会运行，他会拦截掉这个请求，这个东西会走一套流程：
（1）begin，启动一个事务 => TransactionManager
（2）执行目标方法内部的业务逻辑
（3）根据目标方法执行的结果，是否成功，或者是报错，来选择commit / rollback => TransactionManage

## 通过发送url触发执行本地/远程commit、cancel方法
通过读取注解获取commitkey和cancelkey的名字，然后通过ioc容器取出对应的实例，然后通过反射执行对应的commit/cancel方法，利用反射调用cancel组件的方法进行回滚操作method.invoke(instance, args);


## 基于CompensableWork定时组件完成事物中断重启恢复、confirm/cancel执行失败不断重试恢复
