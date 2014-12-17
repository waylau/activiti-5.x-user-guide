##Category 类别

部署和流程定义都是用户定义的类别。流程定义类别在 BPMN 文件中属性的初始化的值 <definitions ... targetNamespace="yourCategory" ...
部署类别是可以直接使用 API 进行指定的看起来想这样：

	repositoryService
	    .createDeployment()
	    .category("yourCategory")
	    ...
	    .deploy();

