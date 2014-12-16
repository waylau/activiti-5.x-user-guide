##Deployment cache configuration 配置部署缓存

所有流程定义都被缓存了（解析之后）避免每次使用前都要访问数据库， 因为流程定义数据是不会改变的。 默认，不会限制这个缓存。如果想限制流程定义缓存，可以添加如下配置

	<property name="processDefinitionCacheLimit" value="10" />

这个配置会把默认的 hashmap 缓存替换成 LRU 缓存，来提供限制。 当然，这个配置的最佳值跟流程定义的总数有关， 实际使用中会具体使用多少流程定义也有关。
也你可以注入自己的缓存实现。这个 bean 必须实现
org.activiti.engine.impl.persistence.deploy.DeploymentCache 接口：

	<property name="processDefinitionCache">
	  <bean class="org.activiti.MyCache" />
	</property>

有一个类似的配置叫 knowledgeBaseCacheLimit 和knowledgeBaseCache， 它们是配置规则缓存的。只
有流程中使用规则任务时才会用到。