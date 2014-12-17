##Generating a process diagram 生成流程图

在部署时没有提供图片的情况下，在 上一节中描述,如果流程定义中包含必要的'图像交换'信息时，Activiti 流程引擎竟会自动生成一个图像。

该资源可以按照部署时 提供流程图片完全相同的方式获取。

![](http://99btgc01.info/uploads/2014/12/deployment.image.generation.png)

如果，因为某种原因，在部署的时候，并不需要或者不必要生成流程定义图片，那么就需要在流程引擎配置的属性中使用isCreateDiagramOnDeploy：

	<property name="createDiagramOnDeploy" value="false" />

现在图就不会生成了

