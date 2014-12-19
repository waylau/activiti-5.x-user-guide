##Creating a ProcessEngine 创建 ProcessEngine

Activiti 流程引擎的配置文件是名为 activiti.cfg.xml 文件。 注意这与使用 Spring 方式创建流程引擎是不一样的。

获得 ProcessEngine 最简单的办法是 使用org.activiti.engine.ProcessEngines 类：

	ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine()

它会在 classpath 下搜索 activiti.cfg.xml， 并基于这个文件中的配置构建引擎。 下面代码展示了实例配置。 后面的章节会给出配置参数的详细介绍。
	
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	  <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
	
	    <property name="jdbcUrl" value="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" />
	    <property name="jdbcDriver" value="org.h2.Driver" />
	    <property name="jdbcUsername" value="sa" />
	    <property name="jdbcPassword" value="" />
	
	    <property name="databaseSchemaUpdate" value="true" />
	
	    <property name="jobExecutorActivate" value="false" />
	    <property name="asyncExecutorEnabled" value="true" />
	    <property name="asyncExecutorActivate" value="false" />
	
	    <property name="mailServerHost" value="mail.my-corp.com" />
	    <property name="mailServerPort" value="5025" />
	  </bean>
	
	</beans>


注意配置 XML 文件其实是一个 Spring  的配置文件。 **但不是说 Activiti 只能用在 Spring 环境
中！** 我们只是利用了 Spring 的解析和依赖注入功能 来构建引擎。

配置文件中使用的 ProcessEngineConfiguration 可以通过编程方式创建。 可以使用不同的 bean id（比如，例子第三行）。

	ProcessEngineConfiguration.createProcessEngineConfigurationFromResourceDefault();
	ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource);
	ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource, String beanName);
	ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream);
	ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName);

也可以不使用配置文件，基于默认创建配置 （参考[各种支持类](http://www.activiti.org/userguide/index.html#configurationClasses)）

	ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
	ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();

所有这些 ProcessEngineConfiguration.createXXX() 方法都返回 ProcessEngineConfiguration，后续可以调整成所需的对象。 在调用 buildProcessEngine() 后， 就会创建一个 ProcessEngine：

	ProcessEngine processEngine = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration()
	  .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE)
	  .setJdbcUrl("jdbc:h2:mem:my-own-db;DB_CLOSE_DELAY=1000")
	  .setAsyncExecutorEnabled(true)
	  .setAsyncExecutorActivate(false)
	  .buildProcessEngine();

