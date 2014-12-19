##Async executor activation 启用 Async executor

AsyncExecutor 是管理线程池的组件，可以触发定时器和异步任务。

默认，AsyncExecutor 是不启用的，由于遗留原因使用的是 JobExecutor。不过建议使用新的 AsyncExecutor 来代替。可以通过定义两个属性

	<property name="asyncExecutorEnabled" value="true" />
	<property name="asyncExecutorActivate" value="true" />

asyncExecutorEnabled 属性启用 Async executor 代替旧的 Job executor。第二个属性 asyncExecutorActivate 指示 Activiti 引擎在启动时启动 Async executor 线程池。