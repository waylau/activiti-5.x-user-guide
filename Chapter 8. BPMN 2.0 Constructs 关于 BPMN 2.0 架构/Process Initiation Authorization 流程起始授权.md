##Process Initiation Authorization 流程起始授权

默认所有人在部署的流程定义上启动一个新流程实例。通过流程初始化授权功能定义的用户和组，web 客户端可以限制哪些用户可以启动一个新流程实例。 注意：Activiti 引擎不会校验授权定义。 这个功能只是为减轻 web 客户端开发者实现校验规则的难度。 设置方法与用户任务用户分配类似。 用户或组可以使用 <activiti:potentialStarter> 标签分配为流程的默
认启动者。下面是一个例子：

	<process id="potentialStarter">
	     <extensionElements>
	       <activiti:potentialStarter>
	         <resourceAssignmentExpression>
	           <formalExpression>group2, group(group3), user(user3)</formalExpression>
	         </resourceAssignmentExpression>
	       </activiti:potentialStarter>
	     </extensionElements>
	   <startEvent id="theStart"/>
	   ...

上面的 XML 中，user(user3) 是直接引用了用户 user3，group(group3) 是引用了组 group3。如果没显示设置，默认认为是群组。 也可以使用 <process> 标签的属性，<activiti:candidateStarterUsers> 和<activiti:candidateStarterGroups>。 下面是一个例子：

	<process id="potentialStarter" activiti:candidateStarterUsers="user1, user2"  
	                                        activiti:candidateStarterGroups="group1">
	      ...

可以同时使用这两个属性。

定义流程初始化授权后，开发者可以使用如下方法获得授权定义。 这些代码可以获得给定的用户可以启动哪些流程定义：

    processDefinitions = repositoryService.createProcessDefinitionQuery().startableByUser("userxxx").list();

也可以获得指定流程定义设置的潜在启动者对应的  identity link。

	identityLinks = repositoryService.getIdentityLinksForProcessDefinition("processDefinitionId");

下面例子演示了如何获得可以启动给定流程的用户列表：

    List<User> authorizedUsers =  identityService().createUserQuery().potentialStarter("processDefinitionId").list();

相同的方式，获得可以启动给定流程配置的群组：

    List<Group> authorizedGroups =  identityService().createGroupQuery().potentialStarter("processDefinitionId").list();
