##External form rendering 外部表单的渲染

该 API 同样也允许你执行 Activiti 流程引擎之外的方式渲染你自己的任务表单。这些步骤说明你可以用你自己的方式对任务表单进行渲染。

本质上，所有需要渲染的表单属性都是通过2个服务方法中的一个进行装配的： StartFormData
FormService.getStartFormData(String processDefinitionId) 和 TaskFormdata
FormService.getTaskFormData(String taskId)

表单属性可以通过 ProcessInstance FormService.submitStartFormData(String processDefinitionId,Map<String,String> properties) 和 void FormService.submitStartFormData(String taskId,Map<String,String> properties)2种方式进行提交。

想要了解更多表单属性如何映射为流程变量，可以查看 第 9.1 节 [“表单属性”](Form properties 表单属性.md)

你可以将任何表单模版资源放进你要部署的业务文档之中（如果你想要将它们按照流程的版本进行存储）。它将会在部署中作为一种可用的资源。你可以使用String
ProcessDefinition.getDeploymentId() 和 InputStream RepositoryService.getResourceAsStream(String
deploymentId, String resourceName); 获取部署上去的表单模版。这样就可以获取你的表单模版定义文件 ，那么你就可以在你自己的应用中渲染/显示你的表单。

你也可以使用该功能获取任务表单之外的其他的部署资源用于其他的目的。
属性`<userTask activiti:formKey="..."`通过 API String FormService.getStartFormData(String
processDefinitionId).getFormKey() 和 String FormService.getTaskFormData(String taskId).getFormKey()暴
露出来的。 你可以使用这个存储你部署的模版中的全名（例如`org/activiti/example/form/mycustom-form.xml`）,但是这并不是必须的。 例如，你可以在表单属性中存储一个通用的 key，然后运用一种算法或者换转去得到你实际使用的模版。当你想要通过不同UI技术渲染不能的表
单，这可能更加方便，例如，使用正常屏幕大小的web应用程序的表单，移动手机小屏幕的表单和甚至可能是 IM 表单和 email 表单模版。