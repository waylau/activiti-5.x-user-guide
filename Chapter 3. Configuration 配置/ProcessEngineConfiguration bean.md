##ProcessEngineConfiguration bean

activiti.cfg.xml 必须包含一个 bean, id为'processEngineConfiguration'。

	<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">

这个 bean 会用来构建 ProcessEngine。 有多个类可以用来定义processEngineConfiguration。 这些类对应不同的环境，并设置了对应的默认值。 最好选择（最）适用于你的环境的类， 这样可以少配置几个引擎的参数。 下面是目前可以使用的类（以后会包含更多）：

* org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration: 单独运行的
流程引擎。Activiti 会自己处理事务。 默认，数据库只在引擎启动时检测 （如果没有 Activiti 的表或者表结构不正确就会抛出异常）。
* org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration: 单元测试时的辅助类。Activiti 会自己控制事务。 默认使用 H2 内存数据库。数据库表会在引擎启动时创建，关闭时删除。 使用它时，不需要其他配置（除非使用 job 执行器或邮件功
能）。
* org.activiti.spring.SpringProcessEngineConfiguration: 在Spring 环境下使用流程引擎。 参考 [Chapter 5. Spring integration 集成 Spring](../Chapter 5. Spring integration 集成 Spring/README.md)。
* org.activiti.engine.impl.cfg.JtaProcessEngineConfiguration: 单独运行流程引擎，并使用 JTA 事务。