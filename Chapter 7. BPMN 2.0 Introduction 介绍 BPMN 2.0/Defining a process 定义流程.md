##Defining a process 定义流程

*注意*

*文章假设你在使用 Eclipse IDE [http://eclipse.org/]来创建和编辑文件。 不过，其中只用到了 Eclipse 很少的特性。你可以使用喜欢的任何工具来创建包含 BPMN 2.0 的 xml 文件。*

创建一个新的 XML 文件（右击任何项目选择“新建”->“其他”->“XML-XML文件”）并命名。 确认文件后缀为 **.bpmn20.xml 或 .bpmn**， 否则引擎无法发布。

![](http://99btgc01.info/uploads/2014/12/new.bpmn.procdef.png)

BPMN 2.0 根节点是 definitions 节点。 这个元素中，可以定义多个流程定义（不过我们建议每个文件只包含一个流程定义， 可以简化开发过程中的维护难度）。 一个空的流程定义看起来像下面这样。注意，definitions 元素 最少也要包含 xmlns 和 targetNamespace 的声明。targetNamespace 可以是任意值，它用来对流程实例进行分类。

	<definitions 
	  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
	  xmlns:activiti="http://activiti.org/bpmn"
	  targetNamespace="Examples">
	
	  <process id="myProcess" name="My First Process">
	    ..
	  </process>
	
	</definitions>

你也可以选择添加线上的 BPMN 2.0 格式位置， 下面是 ecilpse 中的xml 配置。

	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.omg.org/spec/BPMN/20100524/MODEL 
	                    http://www.omg.org/spec/BPMN/2.0/20100501/BPMN20.xsd

process 元素有两个属性：

* id：这个属性是必须的， 它对应着 Activiti ProcessDefinition 对象的 key 属性。id 可以用来启动流程定义的流程实例，通过RuntimeService的startProcessInstanceByKey 方法。这个方法会一直使用最新发布版本的流程定义(译者注：实际中一般都使用这种方式启动流程)。

	ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("myProcess");

注意，它和 startProcessInstanceById 方法不同。 这个方法期望使用Activiti 引擎在发布时自动生成的 id。可以通过调用processDefinition.getId() 方法获得这个值。 生成的 id 的格式为'key:version'， 最大长度限制为64个字符， 如果你在启动时抛出了一个 ActivitiException，说明生成的 id 太长了， 需要限制流程的key的长度。
* name：这个属性是可选的，对应 ProcessDefinition 的 name 属性。 引擎自己不会使用这个属性，它可以用来在用户接口显示便于阅读的名称。


