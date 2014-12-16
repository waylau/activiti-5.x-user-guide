##Automatic resource deployment 资源自动部署

Spring 的集成也有一个专门用于对资源部署的特性。在流程引擎的配置中，你可以指定一组资源。当流程引擎被创建的时候， 所有在这里的资源都将会被自动扫描与部署。在这里有过滤以防止资源重新部署，只有当这个资源真正发生改变的时候，它才会向 Activiti 使用的数据库创建新的部署。 这对于很多用例来说，当 Spring 容器经常重启的情况下（例如测试），使用它是非常不错的选择。

这里有一个例子：

	<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
	  ...
	  <property name="deploymentResources" value="classpath*:/org/activiti/spring/test/autodeployment/autodeploy.*.bpmn20.xml" />
	</bean>
	  
	<bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
	  <property name="processEngineConfiguration" ref="processEngineConfiguration" />
	</bean>

默认，上面的配置会把所有匹配的资源发布到 Activiti 引擎的一个单独发布包下。用来检测防止未修改资源重复发布的机制会作用到整个发布包中。 有时候，这可能不是你想要的。比如，如果你发布了很多流程资源，但是只修改里其中某一个单独的流程定义， 整个发布包都会被认为变更了，导致整个发布包下的所有流程定义都会被重新发布， 结果就是每个流程定义都生成了新版本，虽然其中只有一个流程发生了改变。

为了定制发布方式，你可以为 SpringProcessEngineConfiguration 指定一个额外的参数 deploymentMode。 这个参数指定了匹配多个资源时的发布处理方式。默认下这个参数支持设置三个值：

* default: 把所有资源放在一个单独的发布包中，对这个发布包进行重复检测。 这是默认值，如果你没有指定参数值，就会使用它。
* single-resource: 为每个单独的资源创建一个发布包，并对这些发布包进行重复检测。 你可以单独发布每个流程定义，并在修改流程定义后只创建一个新的流程定义版本。
* resource-parent-folder: 把放在同一个上级目录下的资源发布在一个单独的发布包中，并对发布包进行重复检测。 当需要多资源需要创建发布包，但是需要根据共同的文件夹来组合一些资源时，可以使用它。


这儿有一个例子来演示将 deploymentMode 参数配置为 single-resource 的情况：

	<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
	  ...
	  <property name="deploymentResources" value="classpath*:/activiti/*.bpmn" />
	  <property name="deploymentMode" value="single-resource" />
	</bean>

如果想使用上面三个值之外的参数值，你需要自定义处理发布包的行为。 你可以创建一个 SpringProcessEngineConfiguration 的子类，重写getAutoDeploymentStrategy(String deploymentMode) 方
法。 这个方法中处理了对应 deploymentMode 的发布策略

