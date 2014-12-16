##ProcessEngineFactoryBean

可以把 ProcessEngine 作为一个普通的 Spring bean 进行配置。 类
org.activiti.spring.ProcessEngineFactoryBean 是集成的切入点。 这个 bean 需要一个流程引擎配置来创建流程引擎。这也意味着在文档的[配置这一章](../Chapter 3. Configuration 配置/Creating a ProcessEngine 创建 ProcessEngine.md)的介绍属性的创建和配置对于 Spring 来说也是一样的。对于 Spring 集成的配置和流程引擎 bean 看起来像这样：
	
	<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
	    ...
	</bean>
	  
	<bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
	  <property name="processEngineConfiguration" ref="processEngineConfiguration" />
	</bean>
	  
注意现在 processEngineConfiguration 的 bean 是使用
org.activiti.spring.SpringProcessEngineConfiguration 类。