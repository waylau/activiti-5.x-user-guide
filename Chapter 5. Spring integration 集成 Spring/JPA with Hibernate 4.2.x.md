##JPA with Hibernate 4.2.x

在 Activiti 引擎的 serviceTask 或 listener 中使用 Hibernate 4.2.x JPA 时，需要添加 Spring
ORM 这个额外的依赖。 Hibernate 4.1.x及以下版本是不需要的。应该添加如下依赖：

	<dependency>
	  <groupId>org.springframework</groupId>
	  <artifactId>spring-orm</artifactId>
	  <version>${org.springframework.version}</version>
	</dependency>