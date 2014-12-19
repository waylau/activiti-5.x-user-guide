##Logging 日志

从 Activiti 5.12 开始，SLF4J 被用作日志框架，替换了之前使用java.util.logging。 所有日志（activiti, spring, mybatis等等）都转发给 SLF4J 允许使用你选择的日志实现。

**默认 activiti-engine 依赖中没有提供 SLF4J 绑定的jar， 需要根据你的实际需要使用日志框架。**如果没有添加任何实现 jar，SLF4J 会使用NOP-logger，不使用任何日志，不会发出警告，而且什么日志都不会记录。 可以通过 [http://www.slf4j.org/codes.html#StaticLoggerBinder](http://www.slf4j.org/codes.html#StaticLoggerBinder) 了解这些实现。
使用 Maven，比如使用一个依赖（这里使用 log4j），注意你还需要添加一个 version：
配置

 <dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>

activiti-explorer 和 activiti-rest 应用都使用了 Log4j 绑定。执行所有 activiti-* 模块的单元测试页使用了 Log4j。

**特别提醒如果容器 classpath 中存在 commons-logging**： 为了把spring 日志转发给 SLF4J，需要使用桥接（参考[ http://www.slf4j.org/legacy.html#jclOverSLF4J]( http://www.slf4j.org/legacy.html#jclOverSLF4J)）。 如果你的容器提
供了commons-logging实现，请参考下面网页：[ http://www.slf4j.org/codes.html#release]( http://www.slf4j.org/codes.html#release) 来确保稳定性。
使用 Maven 的实例（忽略版本）：

	<dependency>
	  <groupId>org.slf4j</groupId>
	  <artifactId>slf4j-log4j12</artifactId>
	</dependency>

activiti-explorer 和 activiti-rest 应用都使用了 Log4j 绑定。执行所有 activiti-* 模块的单元测试页使用了 Log4j。

**特别提醒如果容器 classpath 中存在 commons-logging**： 为了把spring 日志转发给 SLF4J，需要使用桥接（参考 <http://www.slf4j.org/legacy.html#jclOverSLF4J>）。 如果你的容器提供了 commons-logging 实现，请参考下面网页：<http://www.slf4j.org/codes.html#release> 来确保稳定性。

使用 Maven 的实例（忽略版本）：

	<dependency>
	  <groupId>org.slf4j</groupId>
	  <artifactId>jcl-over-slf4j</artifactId>
	</dependency>

