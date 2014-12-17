##Providing a process diagram 提供流程图

流程定义的流程图可以被添加到部署中，该流程图将会持久化到 Activiti所使用的数据库中并且可以通过 Activiti 的 API 进行访问。该流程图也可以被用来在 Activiti Explorer 控制台中的流程中进行显示。
如果在我们的类路径下面有一个流程，org/activiti/expenseProcess.bpmn20.xml ，该流程定义有
一个流程 key 'expense'。 以下遵循流程定义图片的命名规范（按照这个特地顺序）：

* 如果在部署时一个图片资源已经存在，它是 BPMN2.0 的 XML 文件名后面是流程定义的 key 并且是一个图片的后缀。那么该图片将被使用。在我们的例子中， 这应该是 org/activiti/expenseProcess.expense.png（或者 jpg/gif）。如果你在一个 BPMN2.0 XML 文件中定义多个流
程定义图片，这种方式更有意义。每个流程定义图片的文件名中都将会有一个流程定义 key。
* 如果并没有这样的图片存在，部署的时候寻找与匹配BPMN2.0 XML 文件的名称的图片资
源。在我们的例子中，这应该是 org/activiti/expenseProcess.png. 注意：这意味着在同一个 BPMN2.0 XML 文件夹中的**每个流程定义**都会有相同的流程定义图片。因此，在每一个 BPMN 2.0 XML 文件夹中仅仅只有一个流程定义，这绝对是不会有问题的。

当使用编程式的部署方式：
	
	repositoryService.createDeployment()
	  .name("expense-process.bar")
	  .addClasspathResource("org/activiti/expenseProcess.bpmn20.xml")
	  .addClasspathResource("org/activiti/expenseProcess.png")
	  .deploy();

接下来，可以通过 API 来获取流程定义图片资源：

	ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
	                                                         .processDefinitionKey("expense")
	                                                         .singleResult();
	  
	  String diagramResourceName = processDefinition.getDiagramResourceName();
	  InputStream imageStream = repositoryService.getResourceAsStream(processDefinition.getDeploymentId(), diagramResourceName);
       