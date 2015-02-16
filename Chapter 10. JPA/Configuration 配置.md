##10.2. Configuration 配置

为了能够使用 JPA 的实体，引擎必须有一个对 `EntityManagerFactory`的引用。 这可以通过配置引用或者提供一个持久化单元名称。作为变量的JPA 实体将会被自动检测并进行相应的处理

下面例子中的配置是使用 jpaPersistenceUnitName：

	<bean id="processEngineConfiguration"
	  class="org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration">
	
	<!-- Database configurations -->
	<property name="databaseSchemaUpdate" value="true" />
	<property name="jdbcUrl" value="jdbc:h2:mem:JpaVariableTest;DB_CLOSE_DELAY=1000" />
	
	<property name="jpaPersistenceUnitName" value="activiti-jpa-pu" />
	<property name="jpaHandleTransaction" value="true" />
	<property name="jpaCloseEntityManager" value="true" />
	
	<!-- job executor configurations -->
	<property name="jobExecutorActivate" value="false" />
	
	<!-- mail server configurations -->
	<property name="mailServerPort" value="5025" />
	</bean>

接下来例子中的配置提供了一个我们自定义的 `EntityManagerFactory`(在这个例子中，使用了 OpenJPA 实体管理器)。注意该代码片段仅仅包含与例子相关的 beans，去掉了其他 beans。OpenJPA 实体管理的完整并可以使用的例子可以在 activiti-spring-examples(`/activiti-spring/src/test/java/org/activiti/spring/test/jpa/JPASpringTest.java`)中找到。

	<bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	  <property name="persistenceUnitManager" ref="pum"/>
	  <property name="jpaVendorAdapter">
	    <bean class="org.springframework.orm.jpa.vendor.OpenJpaVendorAdapter">
	      <property name="databasePlatform" value="org.apache.openjpa.jdbc.sql.H2Dictionary" />
	    </bean>
	  </property>
	</bean>
	
	<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
	  <property name="dataSource" ref="dataSource" />
	  <property name="transactionManager" ref="transactionManager" />
	  <property name="databaseSchemaUpdate" value="true" />
	  <property name="jpaEntityManagerFactory" ref="entityManagerFactory" />
	  <property name="jpaHandleTransaction" value="true" />
	  <property name="jpaCloseEntityManager" value="true" />
	  <property name="jobExecutorActivate" value="false" />
	</bean>

同样的配置也可以在编程式创建一个引擎时完成，例如：

	ProcessEngine processEngine = ProcessEngineConfiguration
	.createProcessEngineConfigurationFromResourceDefault()
	.setJpaPersistenceUnitName("activiti-pu")
	.buildProcessEngine();

配置属性：

* jpaPersistenceUnitName: `使用持久化单元的名称（要确保该持久化单元在类路径下是可用的）。根据该规范，默认的路径是/META-INF/persistence.xml)。要么使用 jpaEntityManagerFactory` 或者`jpaPersistenceUnitName`。
* jpaEntityManagerFactory: `一个实现了javax.persistence.EntityManagerFactory 的 bean 的引用`。它将被用来加载实体并且刷新更新。要么使用jpaEntityManagerFactory 或者jpaPersistenceUnitName。
* jpaHandleTransaction: 在被使用的EntityManager 实例上，该标记表示流程引擎是否需要开始和提交/回滚事物。当使用Java事物API（JTA）时，设置为false。
* jpaCloseEntityManager: `该标记表示流程引擎是否应该关闭从 EntityManagerFactory` 获取的 EntityManager的实例。当EntityManager 是由容器管理的时候需要设置为 false（例如 当使用并不是单一事物作用域的扩展持久化上下文的时候）。