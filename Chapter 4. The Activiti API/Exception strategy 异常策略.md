##Exception strategy 异常策略

Activiti 中的基础异常为org.activiti.engine.ActivitiException，一个非检查异常。 这个异常可以在任何时候被 API 抛出，不过特定方法抛出的“特定”的异常都记录在 [javadocs](http://www.activiti.org/javadocs/index.html)中。 例如，下面的 TaskService：

	/**
	 * Called when the task is successfully executed.
	 * @param taskId the id of the task to complete, cannot be null.
	 * @throws ActivitiObjectNotFoundException when no task exists with the given id.
	 */
	 void complete(String taskId);
    
在上面的例子中，当传入一个不存在的任务的 id 时，就会抛出异常。 同时，javadoc **明确指出 taskId 不能为 null，如果传入 null， 就会抛出 ActivitiIllegalArgumentException**。

我们希望避免过多的异常继承，下面的子类用于特定的场合。 流程引擎和API 调用的其他场合不会使用下面的异常， 它们会抛出一个普通的ActivitiExceptions。

* ActivitiWrongDbException：当 Activiti 引擎发现数据库版本号和引擎版本号不一致时抛出。
* ActivitiOptimisticLockingException：对同一数据进行并发方法并出现乐观锁时抛出。
* ActivitiClassLoadingException：当无法找到需要加载的类或在加载类时出现了错误（比如，JavaDelegate，TaskListener等。)
*  ActivitiObjectNotFoundException：当请求或操作的对应不存在时抛出。
*  ActivitiIllegalArgumentException：这个异常表示调用Activiti API 时传入了一个非法的参数，可能是引擎配置中的非法值，或提供了一个非法制，或流程定义中使用的非法值
*  ActivitiTaskAlreadyClaimedException：当任务已经被认领了，再调用 taskService.claim(...) 就会抛出。