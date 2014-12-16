##Job executor activation 启用 Job executor

JobExecutor 是管理一系列线程的组件，可以触发定时器（也包含后续的异步消息）。 在单元测试场景下，很难使用多线程。因此 API 允许查询(ManagementService.createJobQuery) 和执行 job
(ManagementService.executeJob)，所以 job 可以在单元测试中控制。 要避免与 job 执行器冲突，可以关闭它。

默认，JobExecutor 在流程引擎启动时就会激活。 如果不想在流程引擎启动后自动激活 JobExecutor，可以设置

	<property name="jobExecutorActivate" value="false" />

当流程引擎引导时,不想激活 JobExecutor。

