##Activiti database setup 数据库安装

就像在一分钟版本示例里说过的，Activiti Explorer 默认使用 H2 内存数据库。 要 让Activiti 使用独立运行的 H2 数据库或者其他数据库，可以修改 Activiti Explorer web 应用 WEB-INF/
classes 目录下的 db.properties。

另外，注意 Activiti Explorer 自动生成了演示用的默认用户和群组，流程定义，数据模型。要想禁用这个功能，要修改 WEB-INF 目录下的activiti-standalone-context.xml。 可以使用
下面的 demoDataGenerator bean 定义代码完全禁用安装默认数据。从代码中也可以看出，我们可以单独启用或禁用每一项功能。

	  <bean id="demoDataGenerator" class="org.activiti.explorer.demo.DemoDataGenerator">
	        <property name="processEngine" ref="processEngine" />
	        <property name="createDemoUsersAndGroups" value="false" />
	        <property name="createDemoProcessDefinitions" value="false" />
	        <property name="createDemoModels" value="false" />
	   </bean>
