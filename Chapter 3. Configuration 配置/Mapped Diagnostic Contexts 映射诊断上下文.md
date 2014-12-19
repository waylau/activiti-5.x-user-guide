##Mapped Diagnostic Contexts 映射诊断上下文

在5.13中，activiti 支持 slf4j 的 MDC 功能。 如下的基础信息会传递到日志中记录：

* 流程定义 Id 标记为 mdcProcessDefinitionID
* 流程实例 Id 标记 为mdcProcessInstanceID
* 分支 Id 标记为 mdcexecutionId

默认不会记录这些信息。可以配置日志使用期望的格式来显示它们，扩展通常的日志信息。比如，下面的 log4j 配置定义会让日志显示上面提及的信息：

	log4j.appender.consoleAppender.layout.ConversionPattern =ProcessDefinitionId=%X{mdcProcessDefinitionID}
	executionId=%X{mdcExecutionId} mdcProcessInstanceID=%X{mdcProcessInstanceID} mdcBusinessKey=%X{mdcBusinessKey} %m%n"

当系统进行高风险任务，日志必须严格检查时，这个功能就非常有用，比如要使用日志分析的情况。