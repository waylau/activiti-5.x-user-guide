##Versioning of process definitions 流程定义的版本

BPMN 中并没有版本的概念，没有版本也是不错的，因为可执行的 BPMN 流程作为你开发项目的一部分存在版本控制系统的知识库中（例如 SVN,Git 或者 Mercurial ）。 而在 Activiti 中，流程定义的版本是在部署时创建的。在部署的时候，流程定义被存储到 Activiti 使用的数据
库之前，Activiti 将会自动给 流程定义 分配一个版本号。

对于业务文档中每一个的流程定义，都会通过下列部署执行初始化属性 key, version, name 和 id:

* XML 文件中流程定义（流程模型）的 id 属性被当做是流程定义的 key属性。
* XML文件中的流程模型的 name 属性被当做是流程定义的 name 属性。如果该 name 属性并没有指定，那么 id 属性被当做是 name。
* 带有特定 key 的流程定义在第一次部署的时候，将会自动分配版本号为1，对于之后部署相同 key 的流程定义时候，这次部署的版本号将会设置为比当前最大的版本号大1的值。该 key 属性被用来区别不同的流程定义。
* 流程定义中的id属性被设置为 {processDefinitionKey}:{processDefinitionVersion}:
{generated-id}, 这里的 generated-id 是一个唯一的数字被添加，用于确保在集群环境中缓存的流程定义的唯一性。

看下面示例

	<definitions id="myDefinitions" >
	  <process id="myProcess" name="My important process" >
	    ...     

当部署了这个流程定义之后，在数据库中的流程定义看起来像这样：

Table 6.1. 

<table border="1" summary=""><colgroup><col><col><col><col></colgroup><thead><tr><th>id</th><th>key</th><th>name</th><th>version</th></tr></thead><tbody><tr><td>myProcess:1:676</td><td>myProcess</td><td>My important process</td><td>1</td></tr></tbody></table>

假设我们现在部署用一个流程的最新版本号（例如 改变用户任务），但是流程定义的 id 保持不变。 流程定义表将包含以下列表信息：

Table 6.2. 

<table border="1" summary=""><colgroup><col><col><col><col></colgroup><thead><tr><th>id</th><th>key</th><th>name</th><th>version</th></tr></thead><tbody><tr><td>myProcess:1:676</td><td>myProcess</td><td>My important process</td><td>1</td></tr><tr><td>myProcess:2:870</td><td>myProcess</td><td>My important process</td><td>2</td></tr></tbody></table>

当 runtimeService.startProcessInstanceByKey("myProcess") 方法被调用时，它将会使用流程定义版本号为2的，因为这是最新版本的流程定义。可以说每次流程定义创建流程实例时，都会默认使用最新版本的流程定义。

我们应该创建第二个流程，在 Activiti 中，如下,定义并且部署它，该流程定义会添加到流程定义表中

	<definitions id="myNewDefinitions" >
	  <process id="myNewProcess" name="My important process" >
	    ...    

表格如下：

Table 6.3. 

<table border="1" summary=""><colgroup><col><col><col><col></colgroup><thead><tr><th>id</th><th>key</th><th>name</th><th>version</th></tr></thead><tbody><tr><td>myProcess:1:676</td><td>myProcess</td><td>My important process</td><td>1</td></tr><tr><td>myProcess:2:870</td><td>myProcess</td><td>My important process</td><td>2</td></tr><tr><td>myNewProcess:1:1033</td><td>myNewProcess</td><td>My important process</td><td>1</td></tr></tbody></table>

注意：为何新流程的 key 与我们的第一个流程是不同的。尽管流程定义的名称是相同的（当然，我们应该也是可以改变这一点的），Activiti 仅仅只考虑 id 属性判断流程。因此，新的流程定义部署的版本号为1。

