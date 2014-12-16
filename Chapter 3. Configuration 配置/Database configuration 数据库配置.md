##Database configuration 数据库配置

两种方式配置数据库给 Activiti 引擎使用。首先是定义数据库 JDBC 属性：

* jdbcUrl: 数据库的 JDBC URL
* jdbcDriver: 数据库类型的驱动实现
* jdbcUsername: 数据库连接用户名
* jdbcPassword: 数据库连接密码

基于 JDBC 参数配置的数据库连接 会使用默认的[MyBatis](http://www.mybatis.org/)连接池。 下面的参数可以用来配置连接池（来自MyBatis参数）：

* jdbcMaxActiveConnections: 连接池中处于被使用状态的连接的最大值。默认为10
* jdbcMaxIdleConnections: 连接池中处于空闲状态的连接的最大值
* jdbcMaxCheckoutTime: 连接被取出使用的最长时间，超过时间会被强制回收。 默认为20000（20秒）
* jdbcMaxWaitTime: 这是一个底层配置，让连接池可以在长时间无法获得连接时， 打印一条日志，并重新尝试获取一个连接。（避免因为错误配置导致沉默的操作失败）。 默认为20000（20秒）。

数据配置示例: 

	<property name="jdbcUrl" value="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" /><property name="jdbcDriver" value="org.h2.Driver" /><property name="jdbcUsername" value="sa" /><property name="jdbcPassword" value="" />

也可以使用 javax.sql.DataSource 实现 （比如，[Apache Commons ](http://commons.apache.org/
dbcp/)的 DBCP）：

	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" >  <property name="driverClassName" value="com.mysql.jdbc.Driver" />  <property name="url" value="jdbc:mysql://localhost:3306/activiti" />  <property name="username" value="activiti" />  <property name="password" value="activiti" />  <property name="defaultAutoCommit" value="false" /></bean>      <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">      <property name="dataSource" ref="dataSource" />    ...

注意，Activiti 的发布包中没有这些类。 你要自己把对应的类（比如，从 DBCP 里）放到你的 classpath 下。
无论你使用 JDBC 还是 DataSource 的方式，都可以设置下面的配置：

* databaseType: 一般不用设置，因为可以自动通过数据库连接的元数据获取。 只有自动检测失败时才需要设置。 可能的值有：{h2, mysql, oracle, postgres, mssql, db2}。 **如果没使用默认的H2数据库就必须设置这项**。 这个配置会决定使用哪些创建/删除脚本和查询语句。 参考支持数据库章节 了解支持哪些类型。
* databaseSchemaUpdate: 设置流程引擎启动和关闭时如何处理数据库表。
	* false（默认）：检查数据库表的版本和依赖库的版本， 如果版本不匹配就抛出异常。
	* true: 构建流程引擎时，执行检查，如果需要就执行更新。 如果表不存在，就创建
	* create-drop: 构建流程引擎时创建数据库表， 关闭流程引擎时删除这些表。