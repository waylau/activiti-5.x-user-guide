##Working with the Activiti services 使用 Activiti 服务

像上面介绍的那样，要想操作 Activiti 引擎，需要通过 org.activiti.engine.ProcessEngine 实例暴露的服务。 下面的代码假设你已经拥有了一个可以运行的 Activiti 环境。 你就可以操作一个org.activiti.engine.ProcessEngine。 如果只想简单尝试一下代码， 可以下载或者复制 [Activiti单元测试模板](https://github.com/Activiti/activiti-unit-test-template) ，导入到 IDE 中，把testUserguideCode() 方法添加到 org.activiti.MyUnitTest 中。

这个小例子的最终目标是做一个工作业务流程， 演示公司中简单的请假申请：

![](http://99btgc01.info/uploads/2014/12/api.vacationRequest.png)

###Deploying the process 部署流程

任何与“静态”资源有关的数据（比如流程定义）都可以通过 **RepositoryService**访问。 从概念上讲，所以静态数据都是Activiti的资源内容。

在src/test/resources/org/activiti/test 目录下创建一个新的 xml 文件 VacationRequest.bpmn20.xml（如果不使用单元测试模板，你也可以在任何地方创建）， 内容如下。注意这一章不会解释例子中使用的 xml 结构。 如果有需要可以先阅读[bpmn 2.0章](../Chapter 7. BPMN 2.0 Introduction 介绍 BPMN 2.0/What is BPMN 什么是 BPMN.md)来了解这些。

	<?xml version="1.0" encoding="UTF-8" ?>
	<definitions id="definitions"
	             targetNamespace="http://activiti.org/bpmn20" 
	             xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
	             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	             xmlns:activiti="http://activiti.org/bpmn">
	  
	  <process id="vacationRequest" name="Vacation request">
	  
	    <startEvent id="request" activiti:initiator="employeeName">
	      <extensionElements>
	        <activiti:formProperty id="numberOfDays" name="Number of days" type="long" value="1" required="true"/>
	        <activiti:formProperty id="startDate" name="First day of holiday (dd-MM-yyy)" datePattern="dd-MM-yyyy hh:mm" type="date" required="true" />
	        <activiti:formProperty id="vacationMotivation" name="Motivation" type="string" />
	      </extensionElements>
	    </startEvent>
	    <sequenceFlow id="flow1" sourceRef="request" targetRef="handleRequest" />
	    
	    <userTask id="handleRequest" name="Handle vacation request" >
	      <documentation>
	        ${employeeName} would like to take ${numberOfDays} day(s) of vacation (Motivation: ${vacationMotivation}).
	      </documentation> 
	      <extensionElements>
	         <activiti:formProperty id="vacationApproved" name="Do you approve this vacation" type="enum" required="true">
	          <activiti:value id="true" name="Approve" />
	          <activiti:value id="false" name="Reject" />
	        </activiti:formProperty>
	        <activiti:formProperty id="managerMotivation" name="Motivation" type="string" />
	      </extensionElements>
	      <potentialOwner>
	        <resourceAssignmentExpression>
	          <formalExpression>management</formalExpression>
	        </resourceAssignmentExpression>
	      </potentialOwner>         
	    </userTask>
	    <sequenceFlow id="flow2" sourceRef="handleRequest" targetRef="requestApprovedDecision" />
	    
	    <exclusiveGateway id="requestApprovedDecision" name="Request approved?" />
	    <sequenceFlow id="flow3" sourceRef="requestApprovedDecision" targetRef="sendApprovalMail">
	      <conditionExpression xsi:type="tFormalExpression">${vacationApproved == 'true'}</conditionExpression>
	    </sequenceFlow>
	    
	    <task id="sendApprovalMail" name="Send confirmation e-mail" />
	    <sequenceFlow id="flow4" sourceRef="sendApprovalMail" targetRef="theEnd1" />
	    <endEvent id="theEnd1" />
	    
	    <sequenceFlow id="flow5" sourceRef="requestApprovedDecision" targetRef="adjustVacationRequestTask">
	      <conditionExpression xsi:type="tFormalExpression">${vacationApproved == 'false'}</conditionExpression>
	    </sequenceFlow>
	    
	    <userTask id="adjustVacationRequestTask" name="Adjust vacation request">
	      <documentation>
	        Your manager has disapproved your vacation request for ${numberOfDays} days.
	        Reason: ${managerMotivation}
	      </documentation>
	      <extensionElements>
	        <activiti:formProperty id="numberOfDays" name="Number of days" value="${numberOfDays}" type="long" required="true"/>
	        <activiti:formProperty id="startDate" name="First day of holiday (dd-MM-yyy)" value="${startDate}" datePattern="dd-MM-yyyy hh:mm" type="date" required="true" />
	        <activiti:formProperty id="vacationMotivation" name="Motivation" value="${vacationMotivation}" type="string" />
	        <activiti:formProperty id="resendRequest" name="Resend vacation request to manager?" type="enum" required="true">
	          <activiti:value id="true" name="Yes" />
	          <activiti:value id="false" name="No" />
	        </activiti:formProperty>
	      </extensionElements>
	      <humanPerformer>
	        <resourceAssignmentExpression>
	          <formalExpression>${employeeName}</formalExpression>
	        </resourceAssignmentExpression>
	      </humanPerformer>  
	    </userTask>
	    <sequenceFlow id="flow6" sourceRef="adjustVacationRequestTask" targetRef="resendRequestDecision" />
	    
	    <exclusiveGateway id="resendRequestDecision" name="Resend request?" />
	    <sequenceFlow id="flow7" sourceRef="resendRequestDecision" targetRef="handleRequest">
	      <conditionExpression xsi:type="tFormalExpression">${resendRequest == 'true'}</conditionExpression>
	    </sequenceFlow>
	    
	     <sequenceFlow id="flow8" sourceRef="resendRequestDecision" targetRef="theEnd2">
	      <conditionExpression xsi:type="tFormalExpression">${resendRequest == 'false'}</conditionExpression>
	    </sequenceFlow>
	    <endEvent id="theEnd2" />
	      
	  </process>
	  
	</definitions>
	            
为了让 Activiti 引擎知道这个流程，我们必须先进行“部署”。 部署意味着引擎会把 BPMN 2.0 xml 解析成可以执行的东西， “部署包”中的所有流程定义都会添加到数据库中。 这样，当引擎重启时，它依然可以获得“已部署”的流程：

	ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
	RepositoryService repositoryService = processEngine.getRepositoryService();
	repositoryService.createDeployment()
	  .addClasspathResource("org/activiti/test/VacationRequest.bpmn20.xml")
	  .deploy();
	      
	Log.info("Number of process definitions: " + repositoryService.createProcessDefinitionQuery().count()); 

可以阅读[部署章节](../Chapter 6. Deployment 部署/Business archives 业务文档.md)来了解更多关于部署的信息。

###Starting a process instance 开始流程实例

把流程定义发布到 Activiti 引擎后，我们可以基于它发起新流程实例。 对每个流程定义，都可以有很多流程实例。 流程定义是“蓝图”，流程实例是它的一个运行的执行。

所有与流程运行状态相关的东西都可以通过 **RuntimeService** 获得。 有很多方法可以启动一个新流程实例。在下面的代码中，我们使用定义在流程定义 xml 中的 key 来启动流程实例。 我们也可以在流程实例启动时添加一些流程变量，因为第一个用户任务的表达式需要这些变量。 流程变量经常会被用到，因为它们赋予来自同一个流程定义的不同流程实例的特别含义。 简单来说，流程变量是区分流程实例的关键。

	Map<String, Object> variables = new HashMap<String, Object>();
	variables.put("employeeName", "Kermit");
	variables.put("numberOfDays", new Integer(4));
	variables.put("vacationMotivation", "I'm really tired!");
	      
	RuntimeService runtimeService = processEngine.getRuntimeService();
	ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("vacationRequest", variables);
	      
	// Verify that we started a new process instance
	Log.info("Number of process instances: " + runtimeService.createProcessInstanceQuery().count());

###Completing tasks 完成任务

流程启动后，第一步就是用户任务。这是必须由系统用户处理的一个环节。 通常，用户会有一个“任务列表”，展示了所有须由整个用户处理的任务。 下面的代码展示了对应的查询可能是怎样的：
	
	// Fetch all tasks for the management group
	TaskService taskService = processEngine.getTaskService();
	List<Task> tasks = taskService.createTaskQuery().taskCandidateGroup("management").list();
	for (Task task : tasks) {
	  Log.info("Task available: " + task.getName());
	}            
       
为了让流程实例继续运行，我们需要完成整个任务。对 Activiti 来说，就是需要 complete 任务。 下面的代码展示了如何做这件事：

	Task task = tasks.get(0);
	      
	Map<String, Object> taskVariables = new HashMap<String, Object>();
	taskVariables.put("vacationApproved", "false");
	taskVariables.put("managerMotivation", "We have a tight deadline!");
	taskService.complete(task.getId(), taskVariables);

流程实例会进入到下一个环节。在这里例子中， 下一环节允许员工通过表单调整原始的请假申请。员工可以重新提交请假申请， 这会使流程重新进入到第一个任务。           

###Suspending and activating a process 挂起，激活一个流程

我们可以挂起一个流程定义。当挂起流程定时时， 就不能创建新流程了（会抛出一个异常）。 可以通过 RepositoryService 挂起一个流程

	repositoryService.suspendProcessDefinitionByKey("vacationRequest");
	try {
	  runtimeService.startProcessInstanceByKey("vacationRequest");
	} catch (ActivitiException e) {
	  e.printStackTrace();
	}    

要想重新激活一个流程定义，可以调用repositoryService.activateProcessDefinitionXXX 方法。也可以挂起一个流程实例。挂起时，流程不能继续执行（比如，完成任务会抛出异常）， 异步操作（比如定时器）也不会执行。挂起流程实例可以调用
runtimeService.suspendProcessInstance 方法。 激活流程实例可以调用 runtimeService.activateProcessInstanceXXX 方法。

###Further reading 扩展阅读

上面章节中我们仅仅覆盖了 Activiti 功能的表层。 未来我们会继续扩展这些章节，以覆盖更多 Activiti API。 当然，像其他开源项目一样，学习的最好方式 是研究代码，阅读 Javadocs。

