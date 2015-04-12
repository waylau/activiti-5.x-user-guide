##10.3. Usage 用法

###10.3.1. 简单例子

使用 JPA 变量的例子可以在 JPAVariableTest 中找到。我们将会一步一步的解释 `JPAVariableTest.testUpdateJPAEntityValues`。

首先，我们需要创建一个基于 `META-INF/persistence.xml` 的EntityManagerFactory 作为我们的持久化单元。它包含持久化单元中所有的类和一些供应商特定的配置。

我们将使用一个简单的实体作为测试，其中包含有一个 id 和 `String` 类型的 value 属性，这也将会被持久化。在允许测试之前，我们创建一个实体并且保存它。
	
	@Entity(name = "JPA_ENTITY_FIELD")
	public class FieldAccessJPAEntity {
	
	  @Id
	  @Column(name = "ID_")
	  private Long id;
	
	  private String value;
	
	  public FieldAccessJPAEntity() {
	    // Empty constructor needed for JPA
	  }
	
	  public Long getId() {
	    return id;
	  }
	
	  public void setId(Long id) {
	    this.id = id;
	  }
	
	  public String getValue() {
	    return value;
	  }
	
	  public void setValue(String value) {
	    this.value = value;
	  }
	}

我们开始一个新的流程实例，添加实体作为变量。与其它的变量一样，它们存储在引擎的持久存储区。当下次这个变量被请求，它会从基于类和Id的存储的 `EntityManager` 中加载。

	Map<String, Object> variables = new HashMap<String, Object>();
	variables.put("entityToUpdate", entityToUpdate);
	
	ProcessInstance processInstance = runtimeService.startProcessInstanceByKey("UpdateJPAValuesProcess", variables);

在我们的流程定义的第一个节点包含一个 `serviceTask` 将调用方法 在 `entityToUpdate` 上 `setValue`，它解析为 JPA 变量，我们启动流程实例并将从相关联的当前引擎的上下文“EntityManager”进行加载。

	<serviceTask id='theTask' name='updateJPAEntityTask'
	  activiti:expression="${entityToUpdate.setValue('updatedValue')}" />

当 service-task 完成后，流程实例等在流程定义中定义的 userTask，这使我们能够检查流程实例。在这一点上，EntityManager 已刷新并更改到实体已经被推到数据库。当我们得到变量 entityToUpdate 值时，它再次加载，我们得到实体，并将实体中的属性 `value` 设置到 `updatedValue` 。

	// Servicetask in process 'UpdateJPAValuesProcess' should have set value on entityToUpdate.
	Object updatedEntity = runtimeService.getVariable(processInstance.getId(), "entityToUpdate");
	assertTrue(updatedEntity instanceof FieldAccessJPAEntity);
	assertEquals("updatedValue", ((FieldAccessJPAEntity)updatedEntity).getValue());

###10.3.2. 查询 JPA 处理变量

可以查询 `ProcessInstances` 和 `Execution` 包含 JPA 实体作为变量值。注意 只有 在 ProcessInstanceQuery 和 ExecutionQuery ，`variableValueEquals(name, entity)` 是支持 JPA 实体的 。
方法 `variableValueNotEquals`, `variableValueGreaterThan`, `variableValueGreaterThanOrEqual`, `variableValueLessThan `和 `variableValueLessThanOrEqual` 不支持，并且当一个 JPA 实体传递作为值时，会抛出 `ActivitiException`。

	ProcessInstance result = runtimeService.createProcessInstanceQuery()
	    .variableValueEquals("entityToQuery", entityToQuery).singleResult();

###10.3.3. 高级例子：使用 Spring bean 和 JPA

JPASpringTest 例子可以在 activiti-spring-examples 中找到。它的使用场景如下：


* 存在一个使用 JPA 实体的 Spring bean，用于存储贷款请求。
* 使用 Activiti，我们可以利用那些由现有的 bean 获得的已有的实体，并将其作为变量在流程中使用。
流程是按以下步骤进行定义的：
	* 服务任务，利用已有的 LoanRequestBean 使用启动流程时接收的变量（比如，可以来源于开始的表单）来创建新的 LoanRequest。使用 activiti:resultVariable（它以一个变量来存储表达式结果）将创建出来的实体以变量进行存储。
	* 用户任务，让经理查看请求，批准/不批准，结果存储到一个 boolean 类型的变量 approvedByManager 上。
	* 服务任务，用来更新贷款请求实体以使该实体与流程同步。
	* 根据实体属性 approved 的值，利用排他分支来决定下一步选择哪条路径：当请求被批准时，流程将结束；否则，会另有一个任务（发送拒绝信），这样就可以由拒绝信手动地通知用户了。
	
请注意此流程不包含任何表单，所以其只用于单元测试

![](http://activiti.org/userguide/images/jpa.spring.example.process.png)

	<?xml version="1.0" encoding="UTF-8"?>
	<definitions id="taskAssigneeExample"
	  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
	  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	  xmlns:activiti="http://activiti.org/bpmn"
	  targetNamespace="org.activiti.examples">
	
	  <process id="LoanRequestProcess" name="Process creating and handling loan request">
	    <startEvent id='theStart' />
	    <sequenceFlow id='flow1' sourceRef='theStart' targetRef='createLoanRequest' />
	
	    <serviceTask id='createLoanRequest' name='Create loan request'
	      activiti:expression="${loanRequestBean.newLoanRequest(customerName, amount)}"
	      activiti:resultVariable="loanRequest"/>
	    <sequenceFlow id='flow2' sourceRef='createLoanRequest' targetRef='approveTask' />
	
	    <userTask id="approveTask" name="Approve request" />
	    <sequenceFlow id='flow3' sourceRef='approveTask' targetRef='approveOrDissaprove' />
	
	    <serviceTask id='approveOrDissaprove' name='Store decision'
	      activiti:expression="${loanRequest.setApproved(approvedByManager)}" />
	    <sequenceFlow id='flow4' sourceRef='approveOrDissaprove' targetRef='exclusiveGw' />
	
	    <exclusiveGateway id="exclusiveGw" name="Exclusive Gateway approval" />
	    <sequenceFlow id="endFlow1" sourceRef="exclusiveGw" targetRef="theEnd">
	      <conditionExpression xsi:type="tFormalExpression">${loanRequest.approved}</conditionExpression>
	    </sequenceFlow>
	    <sequenceFlow id="endFlow2" sourceRef="exclusiveGw" targetRef="sendRejectionLetter">
	      <conditionExpression xsi:type="tFormalExpression">${!loanRequest.approved}</conditionExpression>
	    </sequenceFlow>
	
	    <userTask id="sendRejectionLetter" name="Send rejection letter" />
	    <sequenceFlow id='flow5' sourceRef='sendRejectionLetter' targetRef='theOtherEnd' />
	
	    <endEvent id='theEnd' />
	    <endEvent id='theOtherEnd' />
	  </process>
	
	</definitions>

虽然上面的例子很简单，但确实展示出了结合 Spring 和参数化方法表达式来使用 JPA 的强大。此流程根本不需要定制 java
代码（当然了，除 Spring bean 外），大大加速了部署