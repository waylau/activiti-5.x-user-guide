##Database upgrade 数据库升级

在执行更新之前要先备份数据库（使用数据库的备份功能）

默认，每次构建流程引擎时都会进行版本检测。 这一切都在应用启动或Activiti webapp 启动时发生。 如果 Activiti 发现数据库表的版本与依赖库的版本不同， 就会抛出异常。

要升级，你要把下面的配置 放到 activiti.cfg.xml 配置文件里：
 
	<beans ... >
	
	  <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
	    <!-- ... -->
	    <property name="databaseSchemaUpdate" value="true" />    
	    <!-- ... -->
	  </bean>
	
	</beans>

**然后，把对应的数据库驱动放到 classpath 里。** 升级应用的 Activiti依赖。启动一个新版本的 Activiti 指向包含旧版本的数据库。将 databaseSchemaUpdate 设置为 true， Activiti 会自动将
数据库表升级到新版本， 

**当发现依赖和数据库表版本不通过时。**也可以执行更新升级 DDL 语句。 也可以执行数据库脚本，可以在 Activiti 下载页找到。


