##The Process Engine API and services 流程引擎的 API 和服务

引擎 API 是与 Activiti 打交道的最常用方式。 我们从 ProcessEngine 开始， 创建它的很多种方法都已经在 [配置章节](Chapter 3. Configuration 配置/Creating a ProcessEngine 创建 ProcessEngine.md)中有所涉及。 从 ProcessEngine 中，你可以获得很多囊括工作流/BPM 方法的服务。 ProcessEngine 和服务类都是线程安全的。 你可以在整个服务器中仅保持它们的一个引用就可以了。

![](http://99btgc01.info/uploads/2014/12/api.services.png)

	ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
	
	RuntimeService runtimeService = processEngine.getRuntimeService();
	RepositoryService repositoryService = processEngine.getRepositoryService();
	TaskService taskService = processEngine.getTaskService();
	ManagementService managementService = processEngine.getManagementService();
	IdentityService identityService = processEngine.getIdentityService();
	HistoryService historyService = processEngine.getHistoryService();
	FormService formService = processEngine.getFormService();

ProcessEngines.getDefaultProcessEngine() 会在第一次调用时 初始化并创建一个流程引擎，以后再调用就会返回相同的流程引擎。 使用对应的方法可以创建和关闭所有流程引擎：ProcessEngines.init() 和 ProcessEngines.destroy()。

ProcessEngines 会扫描所有 activiti.cfg.xml 和 activiti-context.xml 文件。 对于 activiti.cfg.xml 文件，流程引擎会使用Activiti 的经典方式构建：
ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(inputStream).buildProcessEngine()
对于 activiti-context.xml 文件，流程引擎会使用 Spring 方法构建：先创建一个 Spring 的环境，然后通过环境获得流程引擎。

所有服务都是无状态的。这意味着可以在多节点集群环境下运行 ctiviti，每个节点都指向同一个数据库， 不用担心哪个机器实际执行前端的调用。 无论在哪里执行服务都没有问题。

**RepositoryService** 可能是使用 Activiti 引擎时最先接触的服务。 它提供了管理和控制发布包和流程定义的操作。 这里不涉及太多细节，流程定义是 BPMN 2.0 流程的 Java  实现。 它包含了一个流程每个环节的结构和行为。 发布包是 Activiti 引擎的打包单位。一个发布包可以包含多个 BPMN 2.0 xml 文件和其他资源。 开发者可以自由选择把任意资源包含到发布包中。 既可以把一个单独的 BPMN 2.0 xml 文件放到发布包里，也可以把整个流程和相关资源都放在一起。 （比如，'hr-processes' 实例可以包含hr流程相关的任何资源）。 可以通过 RepositoryService 来部署这种发布包。 发布一个发布包，意味着把它上传到引擎中，所有流
程都会在保存进数据库之前分析解析好。 从这点来说，系统知道这个发布包的存在，发布包中包含的流程就已经可以启动了。

除此之外，服务可以

* 查询引擎中的发布包和流程定义。
* 暂停或激活发布包，对应全部和特定流程定义。 暂停意味着它们不能再执行任何操作了，激活是对应的反向操作。
* 获得多种资源，像是包含在发布包里的文件，或引擎自动生成的流程图。
* 获得流程定义的pojo版本， 可以用来通过java解析流程，而不必通过xml。

正如 RepositoryService 负责静态信息（比如，不会改变的数据，至少是不怎么改变的），**RuntimeService** 正好是完全相反的。它负责启动一个流程定义的新实例。 如上所述，流程定义定义了流程各个节点的结构和行为。 流程实例就是这样一个流程定义的实例。对每个流程定义来说，同一时间会有很多实例在执行。 RuntimeService 也可以用来获取和保存流程变量。 这些数据是特定于某个流程实例的，并会被很多流程中的节点使用 （比如，一个排他网关常常使用流程变量来决定选择哪条路径继续流程）。 Runtimeservice 也能查询流程实例和执行。 执行对应 BPMN 2.0 中的'token'。基本上执行指向流程实例当前在哪里。 最后，RuntimeService 可以在流程实例等待外部触发时使用，这时可以用来继续流程实例。 流程实例可以有很多暂停状态，而服务提供了多种方法来'触发'实例， 接受外部触发后，流程实例就会继续向下执行。

任务是由系统中真实人员执行的，它是 Activiti 这类 BPMN 引擎的核心功能之一。 所有与任务有关的功能都包含在**TaskService**中：

* 查询分配给用户或组的任务
* 创建独立运行任务。这些任务与流程实例无关。
* 手工设置任务的执行者，或者这些用户通过何种方式与任务关联。
* 认领并完成一个任务。认领意味着一个人期望成为任务的执行者， 即这个用户会完成这个任务。完成意味着“做这个任务要求的事情”。 通常来说会有很多种处理形式。

**IdentityService** 非常简单。它可以管理（创建，更新，删除，查询...）群组和用户。 请注意， Activiti 执行时并没有对用户进行检查。 例如，任务可以分配给任何人，但是引擎不会校验系统中是否存在这个用户。 这是 Activiti 引擎也可以使用外部服务，比如 ldap，活动目录，等等。

**FormService** 是一个可选服务。即使不使用它，Activiti 也可以完美运行，不会损失任何功能。这个服务提供了启动表单和任务表单两个概念。 启动表单会在流程实例启动之前展示给用户， 任务表单会在用户完成任务时展示。Activiti 支持在 BPMN 2.0 流程定义中设置这些表单。 这个服务以一种简单的方式将数据暴露出来。再次重申，它是可选的， 表单也不一定要嵌入到流程定义中。

**HistoryService**提供了 Activiti 引擎的所有历史数据。 在执行流程时，引擎会保存很多数据（根据配置），比如流程实例启动时间，任务的参与者， 完成任务的时间，每个流程实例的执行路径，等等。 这个服务主要通过查询功能来获得这些数据。

**ManagementService**在使用 Activiti 的定制环境中基本上不会用到。 它可以查询数据库的表和表的元数据。另外，它提供了查询和管理异步操作的功能。 Activiti 的异步操作用途很多，比如定时器，异步操作， 延迟暂停、激活，等等。后续，会讨论这些功能的更多细节。

可以从[javadocs](http://www.activiti.org/javadocs/index.html)中获得这些服务和引擎 API 的更多信息。