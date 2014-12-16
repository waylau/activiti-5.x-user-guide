##JNDI Datasource Configuration 数据源配置

默认，Activiti 的数据库配置会放在 web 应用的 WEB-INF/classes 目录下的 db.properties 文件中。 这样做比较繁琐， 因为要用户在每次发布时，都修改 Activiti 源码中的 db.properties 并重新编译 war 文件， 或者解压缩 war 文件，修改其中的 db.properties。

使用 JNDI（Java Naming and Directory Interface - Java 命名和目录接口）来获取数据库连接， 连接是由 Servlet 容器管理的，可以
在 war 部署外边管理配置。 与 db.properties 相比， 它也允许对连接进行更多的配置。

###Usage 使用

要想把 Activiti Explorer 和 Activiti Rest 应用从db.properties 转换为使用 JNDI 数据库配
置，需要打开原始的 Spring 配置文件 （activiti-webapp-explorer2/src/main/webapp/WEBINF/activiti-standalone-context.xml 和activiti-webapp-rest2/src/main/resources/
activiti-context.xml）， 删除"dbProperties"和"dataSource"两个bean，然后添加如下
bean：

	<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">    
	<property name="jndiName" value="java:comp/env/jdbc/activitiDB"/>
	</bean>

接下来，我们需要添加包含了默认的 H2 配置的 context.xml 文件。 如果已经有了 JNDI 配置，
会覆盖这些配置。 对 Activiti Explorer 来说，对应的配置文件activiti-webappexplorer2/src/main/webapp/META-INF/context.xml 如下所示：

	<Context antiJARLocking="true" path="/activiti-explorer2">    
	<Resource auth="Container"              name="jdbc/activitiDB"              type="javax.sql.DataSource"              scope="Shareable"              description="JDBC DataSource"              url="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000"              driverClassName="org.h2.Driver"              username="sa"              password=""              defaultAutoCommit="false"              initialSize="5"              maxWait="5000"              maxActive="120"              maxIdle="5"/>
	</Context>

对于 Activiti REST 应用，添加的 activiti-webapp-rest2/src/main/webapp/META-INF/context.xml 如下所示：

	<?xml version="1.0" encoding="UTF-8"?><Context antiJARLocking="true" path="/activiti-rest2">    	
		<Resource auth="Container"              name="jdbc/activitiDB"              type="javax.sql.DataSource"              scope="Shareable"              description="JDBC DataSource"              url="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=-1"              driverClassName="org.h2.Driver"              username="sa"              password=""              defaultAutoCommit="false"              initialSize="5"              maxWait="5000"              maxActive="120"              maxIdle="5"/>
	</Context>

可选的一步，现在可以删除 Activiti Explorer 和 Activiti Rest 两个应用中 不再使用的 db.properties 文件了。

###Configuration 配置

JNDI 数据库配置会因为你使用的 Servlet 容器 不同而不同。 下面的配置可以在 Tomcat 中使用，但是对其他容器，请引用你使用的容器的文档。

如果使用 Tomcat，JNDI资源配置在 
$CATALINA_BASE/conf/[enginename]/[hostname]/[warname].xml （对于 Activiti Explorer 来说，通常是在 $CATALINA_BASE/conf/
Catalina/localhost/activiti-explorer.war）。 当应用第一次发布时，会把这个文件从 war 中复制出来。 所以如果这个文件已经存在了，你需要替换它。要想修改 JNDI 资源让应用连接 mysql 而不是 H2，可以像下面这样修改：

	<?xml version="1.0" encoding="UTF-8"?>    <Context antiJARLocking="true" path="/activiti-explorer2">        
		<Resource auth="Container"            name="jdbc/activitiDB"            type="javax.sql.DataSource"            description="JDBC DataSource"            url="jdbc:mysql://localhost:3306/activiti"            driverClassName="com.mysql.jdbc.Driver"            username="sa"            password=""            defaultAutoCommit="false"            initialSize="5"            maxWait="5000"            maxActive="120"            maxIdle="5"/>        
	</Context>