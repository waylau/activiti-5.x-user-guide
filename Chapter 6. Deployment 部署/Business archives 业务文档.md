##Business archives 业务文档

为了部署流程，它们不得不包装在一个业务文档中。一个业务文档是Activiti 引擎部署的单元。一个业务文档相当与一个压缩文件，它包含BPMN2.0 流程，任务表单，规则和其他任意类型的文件。 大体上，业务文档是包含命名资源的容器。

当一个业务文档被部署，它将会自动扫描以 .bpmn20.xml 或者.bpmn 作为扩展名的 BPMN 文件。每个那样的文件都将会被解析并且可能会包含多个流程定义。

*注意*

*业务归档中的 Java 类将**不能够添加到类路径下**。为了能够让流程运行，必须把存在于业务归档程中的流程定义使用的所有自定义的类（例如：Java服务任务或者实现事件的监听器）放在 activiti 引擎的类路径下*

###Deploying programmatically 编程式部署

通过一个压缩文件部署业务归档，它看起来像这样：
	
	String barFileName = "path/to/process-one.bar";
	ZipInputStream inputStream = new ZipInputStream(new FileInputStream(barFileName));
	    
	repositoryService.createDeployment()
	    .name("process-one.bar")
	    .addZipInputStream(inputStream)
	    .deploy();

它也可以通过一个独立资源构建部署。 详细信息请查看javadocs。

###Deploying with Activiti Explorer 通过 Activiti Explorer 控制台部署

Activiti web 控制台允许你通过 web 界面的用户接口上传一个 bar 格式的压缩文件（或者一个 bpmn20.xml 格式的文件）。 选择 Management 标签,点击 Deployment:

![](http://99btgc01.info/uploads/2014/12/deployment.explorer.png)

现在将会有一个弹出窗口允许你从电脑上面选择一个文件，或者你可以简单的拖拽到指定的区域（如果你的浏览器支持）。

![](http://99btgc01.info/uploads/2014/12/deployment.explorer.2.png)

