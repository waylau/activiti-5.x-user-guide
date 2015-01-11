##Variables 变量

每个流程实例需要并且使用数据来执行存在的步骤。在 Activiti,这些数据称为 variables（变量）,并存储在数据库中。变量可用于表达式(例如在单独的网关中选择正确的流出序列流),在 java 服务任务调用外部服务(例如提供输入或存储服务调用的结果),等等。

一个流程实例可以有变量(称为 process variables 流程变量),但也可以执行(流程是活动的特定指针)并且用户任务可以有变量。一个流程实例可以拥有任意数量的变量。每个变量存储在 ACT_RU_VARIABLE 数据库表中的一行。

任何 startProcessInstanceXXX 方法都有一个可选的参数来提供变量，当流程实例创建和开始时。例如, RuntimeService:

	ProcessInstance startProcessInstanceByKey(String processDefinitionKey, Map<String, Object> variables);

在流程执行时可以添加变量。例如(RuntimeService):

	void setVariable(String executionId, String variableName, Object value);
	void setVariableLocal(String executionId, String variableName, Object value);
	void setVariables(String executionId, Map<String, ? extends Object> variables);
	void setVariablesLocal(String executionId, Map<String, ? extends Object> variables);

注意变量可以设置在本地，对于一个给定的执行(记住一个流程实例由一个执行树组成)。变量只在执行时可见,并且不会高于执行树。当数据不应该传播到流程实例级别,或变量有在流程实例中特定路径的新值(例如当使用并行路径时)，这可能是有用的。

变量也可以再次获取,如下所示。注意,类似的方法在 TaskService 存在。这意味着任务跟执行一样,可以使用局部变量,为了任务的持续时间 而 alive (存活)。
 
	Map<String, Object> getVariables(String executionId);
	Map<String, Object> getVariablesLocal(String executionId);
	Map<String, Object> getVariables(String executionId, Collection<String> variableNames);
	Map<String, Object> getVariablesLocal(String executionId, Collection<String> variableNames);
	Object getVariable(String executionId, String variableName);
	<T> T getVariable(String executionId, String variableName, Class<T> variableClass);

变量 经常使用在 [Java delegates](http://www.activiti.org/userguide/index.html#bpmnJavaServiceTask), [表达式](Expressions 表达式.md), execution- 或者 tasklisteners ,脚本,等等。在这些结构,当前执行或任务对象是可用的,它可以用于变量设置和/或检索。最简单的方法是:

	execution.getVariables();
	execution.getVariables(Collection<String> variableNames);
	execution.getVariable(String variableName);
	
	execution.setVariables(Map<String, object> variables);
	execution.setVariable(String variableName, Object value);

注意,本地变体也可用于上述所有。

历史(和向后兼容的原因),在上面的任何调用,实际上在幕后所有变量将从数据库中获取。这意味着,如果你有10个变量,并通过  getVariable("myVariable") 只有一次,在幕后其他9个将获取和缓存。这不是坏事,因为后续调用不会再接触到数据库。例如,当您的流程定义有三个连续的服务任务(因此有一个数据库事务),使用一个调用来获取所有变量在第一个服务任务的时候，这样可能比在每个服务任务分别获取所需的变量要好。注意,这个应用在获取和设置变量时。

当然,当使用大量的变量或者只是当您想要严格控制数据库查询和交互的时候,这是不合适的。Activiti 5.17 以来,新方法介绍了给一个更严格的控制,通过添加一个可选参数的新方法,告诉引擎是否需要在幕后将所有变量获取并缓存:
 
	Map<String, Object> getVariables(Collection<String> variableNames, boolean fetchAllVariables);
	Object getVariable(String variableName, boolean fetchAllVariables);
	void setVariable(String variableName, Object value, boolean fetchAllVariables);

当参数 fetchAllVariables 为 true 时,上述行为将完全一样:当获取或设置一个变量,所有其他变量将获取和缓存。

然而,当值是 false 时,将使用特定的查询，其他变量将不会获取和缓存。只有当前问题的变量的值将缓存为后续使用。