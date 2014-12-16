##Transactions 事务

我们将会一步一步地解释在 Spring examples 中公布的 SpringTransactionIntegrationTest 下面是我们使用这个例子的Spring 配置文件（你可以在SpringTransactionIntegrationTestcontext.xml找到它）以下展示的部分包括数据源（dataSource）， 事务管理器（transactionManager），流程引擎（processEngine）和 Activiti 引擎服务。

当把数据源（DataSource）传递给 SpringProcessEngineConfiguration （使用"dataSource"属性）
之后，Activiti内部使用了一个org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy
代理来封装传递进来的数据源（DataSource）。这样做是为了确保从数据源（DataSource）获取的SQL连接能够与 Spring 的事物结合在一起发挥得更出色。这意味它不再需要在你的Spring配置中代理数据源（dataSource）了。 然而它仍然允许你传递一个TransactionAwareDataSourceProxy 到SpringProcessEngineConfiguration中。在这个例子中并不会发生多余的包装。

**为了确保在你的 Spring 配置中申明的一个TransactionAwareDataSourceProxy，你不能把使用它的应用交给Spring事物控制的资源。（例如 DataSourceTransactionManager 和
JPATransactionManager 需要非代理的数据源 ）**

	<beans xmlns="http://www.springframework.org/schema/beans" 
	       xmlns:context="http://www.springframework.org/schema/context"
	       xmlns:tx="http://www.springframework.org/schema/tx"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans   http://www.springframework.org/schema/beans/spring-beans.xsd
	                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
	                           http://www.springframework.org/schema/tx      http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">
	
	  <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
	    <property name="driverClass" value="org.h2.Driver" />
	    <property name="url" value="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" />
	    <property name="username" value="sa" />
	    <property name="password" value="" />
	  </bean>
	
	  <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	    <property name="dataSource" ref="dataSource" />
	  </bean>
	  
	  <bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
	    <property name="dataSource" ref="dataSource" />
	    <property name="transactionManager" ref="transactionManager" />
	    <property name="databaseSchemaUpdate" value="true" />
	    <property name="jobExecutorActivate" value="false" />
	  </bean>
	  
	  <bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
	    <property name="processEngineConfiguration" ref="processEngineConfiguration" />
	  </bean>
	  
	  <bean id="repositoryService" factory-bean="processEngine" factory-method="getRepositoryService" />
	  <bean id="runtimeService" factory-bean="processEngine" factory-method="getRuntimeService" />
	  <bean id="taskService" factory-bean="processEngine" factory-method="getTaskService" />
	  <bean id="historyService" factory-bean="processEngine" factory-method="getHistoryService" />
	  <bean id="managementService" factory-bean="processEngine" factory-method="getManagementService" />
	
	...

Spring 配置文件的其余部分包含 beans 和我们将要在这个特有的例子中的配置：

	<beans>  
	  ...
	  <tx:annotation-driven transaction-manager="transactionManager"/>
	
	  <bean id="userBean" class="org.activiti.spring.test.UserBean">
	    <property name="runtimeService" ref="runtimeService" />
	  </bean>
	
	  <bean id="printer" class="org.activiti.spring.test.Printer" />
	
	</beans>

首先使用任意的一种 Spring 创建应用上下文的方式创建其 Spring 应用上下文。在这个例子中你可以使用类路径下面的 XML 资源来配置我们的Spring 应用上下文：

	ClassPathXmlApplicationContext applicationContext = 
	    new ClassPathXmlApplicationContext("org/activiti/examples/spring/SpringTransactionIntegrationTest-context.xml");

或者, 如果它是一个测试的话:

	@ContextConfiguration("classpath:org/activiti/spring/test/transaction/SpringTransactionIntegrationTest-context.xml")

然后我们就可以得到 Activiti 的服务 beans 并且调用该服务上面的方
法。ProcessEngineFactoryBean 将会对该服务添加一些额外的拦截器，在 Activiti 服务上面的方法使用的是 Propagation.REQUIRED 事物语义。所以，我们可以使用 repositoryService 去部署一个流程，如下所示：
	
	RepositoryService repositoryService = (RepositoryService) applicationContext.getBean("repositoryService");
	String deploymentId = repositoryService
	  .createDeployment()
	  .addClasspathResource("org/activiti/spring/test/hello.bpmn20.xml")
	  .deploy()
	  .getId();

其他相同的服务也是同样可以这么使用。在这个例子中，Spring 的事物将会围绕在 userBean.hello() 上 ，并且调用 Activiti 服务的方法也会加入到这个事物中。

	UserBean userBean = (UserBean) applicationContext.getBean("userBean");
	userBean.hello();

这个 UserBean 看起来像这样。记得在上面 Spring bean 的配置中我们把 repositoryService 注入到 userBean 中。
	
	public class UserBean {
	
	  /** injected by Spring */
	  private RuntimeService runtimeService;
	
	  @Transactional
	  public void hello() {
	    // here you can do transactional stuff in your domain model
	    // and it will be combined in the same transaction as 
	    // the startProcessInstanceByKey to the Activiti RuntimeService
	    runtimeService.startProcessInstanceByKey("helloProcess");
	  }
	  
	  public void setRuntimeService(RuntimeService runtimeService) {
	    this.runtimeService = runtimeService;
	  }
	}

